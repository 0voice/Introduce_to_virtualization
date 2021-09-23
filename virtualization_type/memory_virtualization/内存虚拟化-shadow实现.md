# 内存虚拟化-shadow实现

## 1. 虚拟化目的
* 提供给虚机从 零地址开始的连续物理内存空间视图
* 虚机之间隔离及共享内存资源

## 2. 概念阐述
地址空间和物理内存空间：地址空间可以理解为地址域，比如32bit CPU，能访问的地址空间是2 ^ 32 = 4G，这是地址空间，但是我可以只插1G内存。即使插4G内存，有一部分地址空间还要划分给mmio使用，物理内存占用整个地址空间的一部分，它俩并不是一个慨念。在虚拟化环境中，虚机和物理机都有自己的地址空间
* GVA (Guest Virtual Address)，虚机虚拟地址
* GPA(Guest Physical Address)，虚机物理地址
* HVA(Host Virtual Address)，物理机虚拟地址
* HPA(Host Physical Address)，物理机物理地址

## 3. 内存虚拟化软件实现
虚机OS维护的是GVA->GPA的映射，为了实现从客户机物理地址GPA到VMM(Virtual Machine Manager)物理地址HPA，VMM为每个虚机维护了一张从GPA->HPA地映射表。虚机维护的GVA->GPA页表是没有真正写入CR3的，VMM会截获CR3装载指令或TLB操作指令，然后根据上面两张表，VMM为每一个进程维护一个GVA->HPA的页表，并将页表基地址真正写入CR3。这个过程中，也利用了MMU的一大好处，可以将不连续地物理地址空间提供给虚机，并且虚机以为是连续的。通过上面地操作，每个虚机都有了从 零地址开始的连续物理内存空间视图。

### 3.1 影子页表(Shadow Page Table)
虚机维护的虚拟内页表完成了GVA->GPA的映射，如果将该页表的基地址装入CR3中，必然会出现问题。影子页表的解决方法是由VMM维护的影子页表实现GVA->HPA的地址映射。如下图：

![image](https://user-images.githubusercontent.com/87458342/134488906-d181294e-cf4d-47c1-8603-3e78365159eb.png)

虚实物理地址翻译表又由VMM维护，通过这些转换关系，最终提现在影子页表中，并将影子页表装载在CR3中。

影子页表是被物理MMU装载使用的页表，VMM为每个虚机中的每个页表(每个进程都有自己的页表)都维护了一套影子页表，影子页表在地址转换时能够直接将GVA->HPA，不会引入额外的开销。另外，影子页表的结构不一定与虚机内的页表结构相同，比如可以在64位宿主机上运行32位虚机。

刷新TLB场景：

* 在x86平台，写入CR3，如果CR3内容不变，那么写入CR3的这个动作相当于无效整个TLB
* 写入不同的CR3，相当于进程切换，TLB内容也会无效
* 修改页表项，由于页表项被更改了，此时也需要刷新TLB。但此时一般不需要操作整个TLB，一般用INVLPG操作

影子页表的关键就在于VMM如何捕获虚机对虚机内页表的操作。对于CR3写入及INVLPG操作，都属于特权指令，VMM能够捕获，但是虚机如果直接修改虚机内部页表，实现起来比较复杂。

影子页表性能评估

* 时间：大多数情况下，影子页表可以直接将GVA->HPA，所以时间上来讲，虽然与非虚拟化环境有一定差距，不过还算可以
* 空间：需要为每个虚机内部的每个页表都维护一个与之对应的影子页表，开销很大

## 4. KVM shadow实现
这里讨论的是VT-x + shadow page实现方式，对于没有VT-x的情况纯软件内存虚拟化过于久远，不做讨论。

在虚机刚启动时（64位虚机），虚拟内部模式转换过程是 实地址模式(rmode) -> 保护模式(pmode) -> IA32-e模式(x64)，这里面先不去管如何实现模式切换及切换过程中内存虚拟化实现方式的改变，先只关注在IA32-e模式下如何实现内存虚拟化，后面会单独讲解虚机在模式切换过程中具体实现。在正式介绍shadow具体实现前，先简单了解下MMU如何实现映射的（如果不了解MMU映射实现，最好先查些相关资料），为方便理解，后面讨论得映射页大小均为4KB。

INTEL文档中给出的地址：

![image](https://user-images.githubusercontent.com/87458342/134485867-e8844ca2-ffea-4a8f-ad19-79b7c29897c4.png)

整个页表映射结构如下：

![image](https://user-images.githubusercontent.com/87458342/134485938-38645885-a43e-47e5-975f-5751966fdded.png)

对于虚机内部页表实现的功能是GVA->GPA的映射，这个页表基地址是不能写入CR3的，原因如下：

* 每个虚机的内存空间都是从0开始的连续地址，如果都直接写入CR3，必然会冲突。另外，如果虚机可以直接访问到物理机地址，也没有安全性可言
* GPA并不是与HPA直接对应，需要通过KVM维护的映射关系将GPA转换为HPA，写入到shadow页表中去
* 
下面图说明了，虚机内页表和影子页表的关系，为了说明简单，这里不会像上面画出那么多页表项

![image](https://user-images.githubusercontent.com/87458342/134486023-3a8e0196-287e-4b18-960b-cc3eb136bc7d.png)

上图描述了SPT(shadow page table)是如何根据GPT(guest page table)实现映射的，在虚机内部，看到的物理地址都是GPA，前面说了，虚机页表是不能直接写入CR3的，真正生效的是SPT。SPT的目的只有一个，就是将GVA映射成HPA，那么HPA是怎么来的呢？首先遍历虚机页表，获取GVA映射的GPA，然后通过kvm slot，获取GPA对应的HPA，进而在shadow中建立GVA->HPA的映射。对于上图，有以下几点需要注意：

* 为了便于理解，这里虚机和VMM都采用ia32-e模式，页表均为4级，page size 4KB
* shadow页表是在KVM中分配的，并不占用虚机GPA范围内空间，而且对于shadow页表的在VMM中地址也没有限定。shadow的目的就是将GVA映射成HPA
* 由于虚机内每个进程都有自己的页表，所以每个进程都需要维护一套SPT与其GPT对应

### 4.1 shadow页表建立
虚机启动时涉及到处理器模式转换，rmod->pmod->x64，在rmod和pmod模式下，虽然虚机内没有开启页表，但是此时也要为虚机分配一个SPT，只不过此时的SPT没有GPT与之对应，也不需要通过GVA寻找GPA，因为虚机内没有开启分页模式，此时访问的地址都是GPA。在pmode下，虚机内部会初始化虚GPT，然后设置CR3(vm_exit)，开启分页模式。此时，有了GPT，现在需要建立SPT。SPT的建立在mmu_alloc_roots函数中实现，起初SPT只有PML4一级，且PML4E为空。当虚机产生因为产生page fault而vm_exit时，会判断是由于GPT引起的page fault，还是由于SPT引起的page fault，当前只讨论SPT引起的page fault。如果SPT引起page fault，那么就需要walk GPT，找到发生异常地址GVA对应的GPA，在根据GPA拿到HPA，然后在shadow中建立GVA到HPA映射。

上面仅仅讨论了SPT从无到有的过程，其实还有很多情况，比如SPTE的access和dirty位如何同步给GPTE、GPTE清除了DIRTY又如何同步给SPTE、GPTE更改了如何同步给SPTE等等。归根结底，要做的就是虚机页表和shadow页表的同步，最理想情况就是时刻保持同步，假设GPTE改变了映射，那么SPTE也应该改变。SPTE中dirty或access位变了，那么也要同步给GPTE。下面就针对各个情况来讨论页表同步的实现，这里先不讲具体代码实现，先理清处理流程，代码自然就看懂了。

#### 4.1.1 page table entry的同步
这里先不考虑access和dirty的同步，先就看映射关系的变化。依然假定虚机页表已经建立好，但是这里需要注意，虚机的页表也是虚机的物理地址空间，所以对虚机页表访问也需要shadow page table，当前进程页表的初始化是由其它进程在内核态完成的，那么问题来了，最开始页表是怎么初始化的。答案就是在rmod和pmode模式初始化的，也就是前面说得那个只有SPT，没有GPT的状态。现在考虑的是虚机进程页表已经建立好，如何建立shadow页表的，见下图：

![image](https://user-images.githubusercontent.com/87458342/134486327-e7bb4e02-e67e-4d7f-8ecf-e1b7e007fc67.png)

对上图做下简要说明：

1. 初始状态，虚机初始化了页表，不过此时的SPTE为空

2. 当实际访问的过程中，会使用SPTE作为页表，由于此时页表为空，虚机会因为产生page fault而vm_exit，这里简单说下，虚机产生page fault时，是否会vm_exit是由于vmcs中的EXCEPTION_BITMAP域决定的，比如具有EPT功能时，虚机page fault就无需vm_exit。而对于没有EPT情况，是需要vm_exit。当产生vm_exit时，handle_exception中判断，如果是由于page fault引起的，就会调用FNAME(page_fault)来处理，进而处理shadow缺页情况。

3. 上图中每有一个虚机页表，都有一个shadow页表与之对应，而每一个shadow页表都有一个struct kvm_mmu_page来描述，这里有个关键成员就是gfn，这个值就是当前shadow页表对应的虚机页表在guest os中的GPA。具体作用下面会讲到。同时，所有的struct kvm_mmu_page结构会以gfn为键值，维护在mmu_page_hash链式哈希中。

4. SPT一级级建立好，最后一级的SPTE中对应的就是GPA对应的HPA，这样便可以访问了

上面讲述了当shadow中从无到有的过程，但是GPT在虚机内可能是变化的，所以就涉及GPT和SPT的同步，现在讲述几个同步case

Case1：

针对一些没有SPT与之对应的GPT，如果GPT被修改了，此时如何处理。结论就是不需要管，结合下图说明：

![image](https://user-images.githubusercontent.com/87458342/134486442-fc55c334-ec83-4115-afbd-9f17714a13b6.png)

首先，对于guest os，一旦开启分页模式后，访问任何物理地址都需要经过MMU页表翻译，包括页表本身。图中我现在要修改粉色的页表(标注为"修改页表")，简称粉页表，也就是说要写粉页表对应的GPA。在修改粉页表的过程中，是通过其它页表来访问的粉页表，其它页表需要shadow page table，而粉页表此时可以没有shadow page table，针对这种情况，即使粉页表修改了，也可以不需要同步，因为本身没有shadow page table，也没有同步目标啊。由于没有SPT，所以当需要访问粉页表对应的GPA时，必然page fault，此时会建立shadow page table，这时自然会根据粉页表最新内容创建SPT。注：所谓的同步，就是在不断处理SPT和GPT之间的关系，保证两者表述一致

case2：

就是SPT和GPT本身是同步的，但是GPT修改了，如何同步给SPT。结合下图说明：

![image](https://user-images.githubusercontent.com/87458342/134486663-c1b51d45-fd2e-4675-b731-ad91ae9c5537.png)

下面简述下这个图描述的整个流程：
1.首先，对于guest os，一旦开启分页模式后，访问任何物理地址都需要经过MMU页表翻译，包括页表本身。图中我现在要修改粉色的页表(标注为"修改页表")，简称粉页表，也就是说要写粉页表对应的GPA。所以，此时我需要建立能够访问到粉页表HPA的shadow page table，在建立的最后的spte时，会做一个检查，如果这个粉页表的GPA和mmu_page_hash表中的某一个struct kvm_mmu_page成员的gfn相等，那就说明现在我要访问的GPA是一个虚机页表，并且已经有shadow page table和这个GPA页表关联上，那么既然GPA修改了，对应的shadow page table也要修改，所以struct kvm_mmu_page的unsync会置位，用于后面同步使用。总结一下就是说，在建立shadow page table时候，会看虚机访问的GPA是否和mmu_page_hash表中的某一个struct kvm_mmu_page成员的gfn相等，那么就需要置位unsync。因为相等的话，说明存在shadow page table和guset page table关联着，需要同步。如果mmu_page_hash表中不存在任何struct kvm_mmu_page成员的gfn与该GPA相等，那么即使要访问的GPA是虚机页表，也不存在shadow page table，所以不需要同步，此时和case1场景一样。对于代码中的一些写保护设定，也是为了及时设置unsync状态。这里面讨论的是最后一级页表，对于非最后一级的页表一直都是写保护的，处理方式比上面描述的简单，读者直接看代码即可。

#### 4.1.2 access、dirty位同步
针对SPTE和GPTE中access、dirty的同步。无论是虚机kernel、还是vmm kernel，LRU算法会根据access和dirty标记进行物理页管理。所以当SPTE、和GPTE出现不同步时，要及时同步。造成不同步的原因主要有以下两种：

* 问题1

因为虚机实际使用的是SPTE页表，所以当访问page或者写page时，硬件会自动对SPTE中的access和dirty置位，而虚机在做LRU算法时，依赖与GPTE中的access和dirty位，所以需要将SPTE同步给GPTE

针对上面的问题，access和dirty实现方式略有不同。但是同步时机都是相同的，对于SPTE同步给GPTE，都是在FNAME(page_fault)中实现。先看dirty，上面4.1中讲了，shadow最开始时候是空的，如果虚机执行了写操作，且GPT已经初始化好了，后面在讨论问题时，都假设GPT已经初始化好了(因为虚机本身page fault的处理和虚拟化实现关系不大)，由于shadow为空，此时写操作必然page fault，然后为其建立shadow页表，同时，将access、dirty同步到GPTE。如果虚机读操作，那么会将SPTE设置为写保护，同时GPTE access置位。

* 问题2

虚机会对GPTE中的access和dirty做清除操作，此时也需要将SPTE感知（为了还原到问题1的场景进而对GPTE再次置位）

当虚机内部清除access、dirty时，会调用INVLPG，此时虚机会vm_exit，在FNAME(invlpg)中会根据会清除spte，清除spte保证一个原则，那就是可以还原到问题1中的状态，能够为再次同步GPTE做准备。为了加速访问，这里会进行预取操作，如果GPTE access为1，那么会进行预取，dirty为0的话，那么会将SPTE设置为只读。如果GPTE access为0，则不预取了，为了产生page fault而同步access。

针对上面两个问题，主要考虑两点，一个是同步时机、另一个是同步方法。同步时机两个，分别是FNAME(page_fault)，FNAME(invlpg)。同步方法对于access和dirty略有不通，dirty位可以通过写保护同步，而同步access需要清空SPTE实现。



原文作者： hx_op








