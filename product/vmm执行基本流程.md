# vmm执行基本流程
```C
`vmm_test_begin(testcase_name,vmm_env,"Test Case Name String") ;
env.build() ;
env.reset_dut() ;
env.start() ;
env.wait_for_end() ;
env.report() ;
`vmm_test_end(testcase_name);
```

参考文献：http://www.testbench.in/VM_09_VMM_TEST.html
vmm_test is introduced in vmm 1.1. 

To know the vmm version which you are using, use this command 
vcs -R -sverilog -ntb_opts dtm 
+incdir+$VMM_HOME/sv $VMM_HOME/sv/vmm_versions.sv 

vmm_test is used for compiling all the testcases in one compilation. The simulation of each testcase is done individually. Traditionally for each testcase, compile and simulation are done per testcase file. With this new approach, which dose compilation only once, will save lot of cup. 
Generally each testcase can be divided into two parts. 

## (S)Procedural code part. 
The procedural code part (like passing the above new constrained transaction definition to some atomic generator, calling the env methods etc) has to be defined between these macros. vmm provides 2 macros to define testcase procedural part. 
```C
`vmm_test_begin(testcase_name,vmm_env,"Test Case Name String") 
`vmm_test_env(testcase_name) 
```

## (S)Declarative code part. 
The declarative part( like new constrained transacting definition) is defined outside these macros. 

Writing A Testcase 

Let us see an example of writing a testcase. 
Inside the testcase, we will define a new transaction and pass this transaction to the atomic generator. 


## (S) Declarative part: 
1) Define all the declarative code. 

```C
class constrained_tran extends pcie_transaction; 

// Add some constraints 
// Change some method definitions 

end class 
```

2) Define a handle to the above object. 

constrained_tran c_tran_obj; 

## (S) Procedural part: 

Use a `vmm_test_begin . There are 3 arguments to macro. 
The first argument is the name of the testcase class and will also be used as the name of the testcase in the global testcase registry. 
The second argument is the name of the environment class that will be used to execute the testcase. A data member of that type named "env" will be defined and assigned, ready to be used. 
The third argument is a string documenting the purpose of the test. 

```C
`vmm_test_begin(test_1,vmm_env,"Test_1") 
```

In this testcase, we want to pass the c_tran_obj object. The steps t pass the c_tran_obj as per the vmm_env is 

```C
env.build(); 
c_tran_obj = new(" "); 
env.atomic_gen.randomized_obj = c_tran_obj; 
```

Start the vmm_env execution. 

```C
env.run(); 
```

Now use `vmm_test_end to define the end of the testcase. The argument to this is the testcase name. 

```C
`vmm_test_end(test_1) 
```

Following is the full testcase source code which we discussed above. 

## (S) Testcase source code 

```C
class constrained_tran extends pcie_transaction; 

end class 
```

constrained_tran c_tran_obj 

```C
`vmm_test_begin(test_1,vmm_env,"Test_1") 
$display(" Start of Testcase : Test_1 "); 
env.build(); 
c_tran_obj = new(" "); 
env.atomic_gen.randomized_obj = c_tran_obj; 
env.run(); 
$display(" End of Testcase : Test_1 "); 
`vmm_test_end(test_1) 
```

Like this you can write many testcases in separate file or single file. 

Example Of Using Vmm_test 


Now we will implement 3 simple testcases and a main testcase which will be used for selecting any one of the 3 testcases. 


## (S) testcase_1 

Write this testcase in a testcase_1.sv file 

```C
// Define all the declarative code hear. I done have anything to show you. 
// 
// class constrained_tran extends pcie_transaction; 
// 
// end class 
// 
// constrained_tran c_tran_obj 

`vmm_test_begin(test_1,vmm_env,"Test_1") 
$display(" Start of Testcase : Test_1 "); 
// This is procedural part. You can call build method. 
env.build(); 

// You can pass the above created transaction to atomic gen. 
// c_tran_obj = new(" "); 
// env.atomic_gen.randomized_obj = c_tran_obj; 
// 
// 

env.run(); 
$display(" End of Testcase : Test_1 "); 
`vmm_test_end(test_1) 
```

## (S)testcase_2 
Write this testcase in a testcase_2.sv file 

```C
`vmm_test_begin(test_2,vmm_env,"Test_2") 
$display(" Start of Testcase : Test_2 "); 
// Do something like this .... 
env.build(); 
// like this .... 
env.run(); 
$display(" End of Testcase : Test_2 "); 
`vmm_test_end(test_2) 
```

## (S)testcase_3 

Write this testcase in a testcase_3.sv file 

```C
`vmm_test_begin(test_3,vmm_env,"Test_3") 
$display(" Start of Testcase : Test_3 "); 
// Do something like this .... 
env.build(); 
// like this .... 
env.run(); 
$display(" End of Testcase : Test_3 "); 
`vmm_test_end(test_3) 
```

## (S)main testcase 

Now we will write the main testcase. This doesn't have any test scenario, it is only used for handling the above 3 testcases. 

This should be written in program block. 

1) First include all the above testcases inside the program block. 
```C
`include "testcase_1.sv" 
`include "testcase_2.sv" 
`include "testcase_3.sv" 
```

2) Define env class object. 
```C
vmm_env env = new(); 
```

As I dont have a custom env class to show, I used vmm_env. You can use your custom defined env. 

3) In the initial block call the run() method of vmm_test_registry class and pass the above env object as argument. This run method of vmm_test_registry is a static method, so there is no need to create an object of this class. 

```C
vmm_test_registry::run(env); 
```

## (S) Main testcase source code 
```C
`include "vmm.sv" 
program main(); 

`include "testcase_1.sv" 
`include "testcase_2.sv" 
`include "testcase_3.sv" 

vmm_env env; 

initial 
begin 
$display(" START OF TEST CASE "); 
env = new(); 
vmm_test_registry::run(env); 
$display(" START OF TEST CASE "); 
end 


endprogram 
```

Now compile the above code using command. 

vcs -sverilog main_testcase.sv +incdir+$VMM_HOME/sv 

Now simulate the above compiled code. 

To simulate , just do ./simv and see the log. 
You can see the list of the testcases. This simulation will not ececute any testcase. 

## (S)LOG 

START OF TEST CASE 
*FATAL*[FAILURE] on vmm_test_registry(class) at 0: 
No test was selected at runtime using +vmm_test=<test>. 
Available tests are: 
0) Default : Default testcase that simply calls env::run() 
1) test_1 : Test_1 
2) test_2 : Test_2 
3) test_3 : Test_3 

To run testcase_1 use ./simv +vmm_test=test_1 

## (S)LOG 

START OF TEST CASE 
Normal[NOTE] on vmm_test_registry(class) at 0: 
Running test "test_1"... 
Start of Testcase : Test_1 
Simulation PASSED on /./ (/./) at 0 (0 warnings, 0 demoted errors & 0 demoted warnings) 
End of Testcase : Test_1 
START OF TEST CASE 

Simillarly to run testcase_2 ./simv +vmm_test=test_2 


## (S)LOG 

START OF TEST CASE 
Normal[NOTE] on vmm_test_registry(class) at 0: 
Running test "test_2"... 
Start of Testcase : Test_2 
Simulation PASSED on /./ (/./) at 0 (0 warnings, 0 demoted errors & 0 demoted warnings) 
End of Testcase : Test_2 
START OF TEST CASE 

You can also call the Default testcase, which just calls the env::run() 

./simv +vmm_test=Default 

## (S)LOG 

START OF TEST CASE 
Normal[NOTE] on vmm_test_registry(class) at 0: 
Running test "Default"... 
Simulation PASSED on /./ (/./) at 0 (0 warnings, 0 demoted errors & 0 demoted warnings) 
START OF TEST CASE 

## (S) Download the source code 

vmm_test.tar 
Browse the code in vmm_test.tar 

## (S) Command to compile 
vcs -sverilog -ntb_opts dtm +incdir+$VMM_HOME/sv main_testcase.sv 

## (S) Commands to simulate testcases 
./simv 
./simv +vmm_test=Default 
./simv +vmm_test=test_1 
./simv +vmm_test=test_2 
./simv +vmm_test=test_3 




















