---
title: error-saleorder
tags:
  - idea
categories:
  - 异常
abbrlink: 3282261323
date: 2017-12-28 15:17:58
---
```
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGSEGV (0xb) at pc=0x000000010146e882, pid=65032, tid=0x0000000000005303
#
# JRE version: Java(TM) SE Runtime Environment (8.0_151-b12) (build 1.8.0_151-b12)
# Java VM: Java HotSpot(TM) 64-Bit Server VM (25.151-b12 mixed mode bsd-amd64 compressed oops)
# Problematic frame:
# V  [libjvm.dylib+0x534882]  Symbol::decrement_refcount()+0x4
#
# Failed to write core dump. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again
#
# An error report file with more information is saved as:
# /Users/victor/code/egenieProjects/ejlerp-saleorder/hs_err_pid65032.log
#
# If you would like to submit a bug report, please visit:
#   http://bugreport.java.com/bugreport/crash.jsp
#

Process finished with exit code 134 (interrupted by signal 6: SIGABRT)


```
<!-- more -->

/Users/victor/code/egenieProjects/ejlerp-saleorder/hs_err_pid65032.log
中的文件:
```

#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGSEGV (0xb) at pc=0x000000010146e882, pid=65032, tid=0x0000000000005303
#
# JRE version: Java(TM) SE Runtime Environment (8.0_151-b12) (build 1.8.0_151-b12)
# Java VM: Java HotSpot(TM) 64-Bit Server VM (25.151-b12 mixed mode bsd-amd64 compressed oops)
# Problematic frame:
# V  [libjvm.dylib+0x534882]  Symbol::decrement_refcount()+0x4
#
# Failed to write core dump. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again
#
# If you would like to submit a bug report, please visit:
#   http://bugreport.java.com/bugreport/crash.jsp
#

---------------  T H R E A D  ---------------

Current thread (0x00007fae5502f800):  VMThread [stack: 0x000070000d688000,0x000070000d788000] [id=21251]

siginfo: si_signo: 11 (SIGSEGV), si_code: 1 (SEGV_MAPERR), si_addr: 0x00007f2e56b22522

Registers:
RAX=0x000000010c14e578, RBX=0x000000010c5c9148, RCX=0x0000000000330006, RDX=0x000000000000060a
RSP=0x000070000d787430, RBP=0x000070000d787430, RSI=0x0000000000000000, RDI=0x00007f2e56b22520
R8 =0x0000000000000007, R9 =0x00007fae56abe9c0, R10=0x000007fae56abee1, R11=0x0000000000000008
R12=0x00000000000007d0, R13=0x000000010c5c9148, R14=0x000000000000034d, R15=0x0000000000000001
RIP=0x000000010146e882, EFLAGS=0x0000000000010246, ERR=0x0000000000000004
  TRAPNO=0x000000000000000e

Top of Stack: (sp=0x000070000d787430)
0x000070000d787430:   000070000d787450 000000010111c897
0x000070000d787440:   000000010c5c9148 000000010c5c9148
0x000070000d787450:   000070000d787470 000000010112057c
0x000070000d787460:   00007fae54611360 000000010c5c9148
0x000070000d787470:   000070000d787490 00000001011205f9
0x000070000d787480:   000000010c5c9148 00007fae54611360
0x000070000d787490:   000070000d7874c0 00000001010c4656
0x000070000d7874a0:   0000000000000003 0000000101518ec8
0x000070000d7874b0:   00007fae54611360 0000000000000003
0x000070000d7874c0:   000070000d787500 00000001010c8e64
0x000070000d7874d0:   000000010137f91e 00007fae54611360
0x000070000d7874e0:   000000010182d590 0000000000000000
0x000070000d7874f0:   000000010182d501 00007fae54502d10
0x000070000d787500:   000070000d787520 00000001010c8ec6
0x000070000d787510:   0000000000000000 0000000000000000
0x000070000d787520:   000070000d787550 00000001010c8f50
0x000070000d787530:   000070000d787550 00000001010c831b
0x000070000d787540:   000000010182d501 000000010182d501
0x000070000d787550:   000070000d787590 00000001010c8ff2
0x000070000d787560:   0000000000000000 000000010182d590
0x000070000d787570:   0000000000000000 00007fae553f3ce0
0x000070000d787580:   000000010182d590 000070000d7876e0
0x000070000d787590:   000070000d7875b0 0000000101474c30
0x000070000d7875a0:   000070000d787690 000000010182d590
0x000070000d7875b0:   000070000d787760 00000001013ff48b
0x000070000d7875c0:   000000010182d810 000000000000001c
0x000070000d7875d0:   0000000000000000 00000000000003a5
0x000070000d7875e0:   0000000000000172 0000000000000011
0x000070000d7875f0:   000000010151e9b3 0000000000000080
0x000070000d787600:   000070000d787640 00007fff6764d562
0x000070000d787610:   000070000d787728 0000000000000000
0x000070000d787620:   00000000000003a5 0000000000000172 

Instructions: (pc=0x000000010146e882)
0x000000010146e862:   4c 89 e6 e8 c8 d0 f5 ff 49 ff c7 0f b7 03 41 39
0x000000010146e872:   c7 7c e2 48 8d 35 25 da 0a 00 eb a5 55 48 89 e5
0x000000010146e882:   66 83 7f 02 00 79 02 5d c3 48 83 c7 02 5d e9 f7
0x000000010146e892:   74 b9 ff 90 55 48 89 e5 66 83 7f 02 00 79 02 5d 

Register to memory mapping:

RAX=0x000000010c14e578 is pointing into metadata
RBX=0x000000010c5c9148 is pointing into metadata
RCX=0x0000000000330006 is an unknown value
RDX=0x000000000000060a is an unknown value
RSP=0x000070000d787430 is an unknown value
RBP=0x000070000d787430 is an unknown value
RSI=0x0000000000000000 is an unknown value
RDI=0x00007f2e56b22520 is an unknown value
R8 =0x0000000000000007 is an unknown value
R9 =0x00007fae56abe9c0 is an unknown value
R10=0x000007fae56abee1 is an unknown value
R11=0x0000000000000008 is an unknown value
R12=0x00000000000007d0 is an unknown value
R13=0x000000010c5c9148 is pointing into metadata
R14=0x000000000000034d is an unknown value
R15=0x0000000000000001 is an unknown value


Stack: [0x000070000d688000,0x000070000d788000],  sp=0x000070000d787430,  free space=1021k
Native frames: (J=compiled Java code, j=interpreted, Vv=VM code, C=native code)
V  [libjvm.dylib+0x534882]  Symbol::decrement_refcount()+0x4
V  [libjvm.dylib+0x1e2897]  ConstantPool::unreference_symbols()+0x29
V  [libjvm.dylib+0x1e657c]  ConstantPool::release_C_heap_structures()+0x12
V  [libjvm.dylib+0x1e65f9]  ConstantPool::deallocate_contents(ClassLoaderData*)+0x51
V  [libjvm.dylib+0x18a656]  void MetadataFactory::free_metadata<ConstantPool*>(ClassLoaderData*, ConstantPool*)+0x44
V  [libjvm.dylib+0x18ee64]  ClassLoaderData::free_deallocate_list()+0x96
V  [libjvm.dylib+0x18eec6]  ClassLoaderDataGraph::free_deallocate_lists()+0x1a
V  [libjvm.dylib+0x18ef50]  ClassLoaderDataGraph::clean_metaspaces()+0x5c
V  [libjvm.dylib+0x18eff2]  ClassLoaderDataGraph::do_unloading(BoolObjectClosure*, bool)+0x90
V  [libjvm.dylib+0x53ac30]  SystemDictionary::do_unloading(BoolObjectClosure*, bool)+0x12
V  [libjvm.dylib+0x4c548b]  PSParallelCompact::marking_phase(ParCompactionManager*, bool, ParallelOldTracer*)+0x52f
V  [libjvm.dylib+0x4c6977]  PSParallelCompact::invoke_no_policy(bool)+0x41b
V  [libjvm.dylib+0x4c6f8b]  PSParallelCompact::invoke(bool)+0x57
V  [libjvm.dylib+0x19eef6]  CollectedHeap::collect_as_vm_thread(GCCause::Cause)+0x5e
V  [libjvm.dylib+0x5b2cb4]  VM_CollectForMetadataAllocation::doit()+0xb0
V  [libjvm.dylib+0x5b9b29]  VM_Operation::evaluate()+0x4f
V  [libjvm.dylib+0x5b8195]  VMThread::evaluate_operation(VM_Operation*)+0xdf
V  [libjvm.dylib+0x5b85e2]  VMThread::loop()+0x328
V  [libjvm.dylib+0x5b7f01]  VMThread::run()+0x79
V  [libjvm.dylib+0x48bbb2]  java_start(Thread*)+0xf6
C  [libsystem_pthread.dylib+0x36c1]  _pthread_body+0x154
C  [libsystem_pthread.dylib+0x356d]  _pthread_body+0x0
C  [libsystem_pthread.dylib+0x2c5d]  thread_start+0xd

VM_Operation (0x0000700016e3fbd8): CollectForMetadataAllocation, mode: safepoint, requested by thread 0x00007fae55a24000


---------------  P R O C E S S  ---------------

Java Threads: ( => current thread )
  0x00007fae5638a000 JavaThread "JMX server connection timeout 320" daemon [_thread_blocked, id=128775, stack(0x0000700016838000,0x0000700016938000)]
  0x00007fae55a24000 JavaThread "RMI TCP Connection(10)-192.168.0.123" daemon [_thread_blocked, id=88323, stack(0x0000700016d47000,0x0000700016e47000)]
  0x00007fae55353000 JavaThread "JMX server connection timeout 317" daemon [_thread_blocked, id=129027, stack(0x0000700016c44000,0x0000700016d44000)]
  0x00007fae55808800 JavaThread "RMI TCP Connection(9)-192.168.0.123" daemon [_thread_blocked, id=129283, stack(0x0000700016b41000,0x0000700016c41000)]
  0x00007fae55352800 JavaThread "RMI TCP Connection(8)-127.0.0.1" daemon [_thread_in_native, id=129795, stack(0x0000700016a3e000,0x0000700016b3e000)]
  0x00007fae5638f000 JavaThread "RMI TCP Connection(7)-127.0.0.1" daemon [_thread_in_native, id=130051, stack(0x000070001693b000,0x0000700016a3b000)]
  0x00007fae54c28800 JavaThread "DestroyJavaVM" [_thread_blocked, id=4611, stack(0x000070000d073000,0x000070000d173000)]
  0x00007fae5534f000 JavaThread "Thread-149" [_thread_blocked, id=87315, stack(0x0000700016735000,0x0000700016835000)]
  0x00007fae54c23000 JavaThread "RMI TCP Connection(6)-192.168.0.123" daemon [_thread_in_native, id=65027, stack(0x0000700016632000,0x0000700016732000)]
  0x00007fae5532b000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-41" daemon [_thread_blocked, id=65539, stack(0x000070001652f000,0x000070001662f000)]
  0x00007fae56310800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-40" daemon [_thread_blocked, id=66051, stack(0x000070001642c000,0x000070001652c000)]
  0x00007fae5532a000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-39" daemon [_thread_blocked, id=64259, stack(0x0000700016329000,0x0000700016429000)]
  0x00007fae55cc4000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-38" daemon [_thread_blocked, id=63747, stack(0x0000700016226000,0x0000700016326000)]
  0x00007fae55335800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-37" daemon [_thread_blocked, id=66567, stack(0x0000700016123000,0x0000700016223000)]
  0x00007fae54c22000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-36" daemon [_thread_blocked, id=63235, stack(0x0000700016020000,0x0000700016120000)]
  0x00007fae54c21800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-35" daemon [_thread_blocked, id=66819, stack(0x0000700015f1d000,0x000070001601d000)]
  0x00007fae56502800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-34" daemon [_thread_blocked, id=67075, stack(0x0000700015e1a000,0x0000700015f1a000)]
  0x00007fae55cc3000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-33" daemon [_thread_blocked, id=62467, stack(0x0000700015d17000,0x0000700015e17000)]
  0x00007fae562fc000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-32" daemon [_thread_blocked, id=67587, stack(0x0000700015c14000,0x0000700015d14000)]
  0x00007fae55bd9000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-31" daemon [_thread_blocked, id=67843, stack(0x0000700015b11000,0x0000700015c11000)]
  0x00007fae55335000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-30" daemon [_thread_blocked, id=68359, stack(0x0000700015a0e000,0x0000700015b0e000)]
  0x00007fae562fa000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-29" daemon [_thread_blocked, id=61443, stack(0x000070001590b000,0x0000700015a0b000)]
  0x00007fae54f20800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-28" daemon [_thread_blocked, id=68611, stack(0x0000700015808000,0x0000700015908000)]
  0x00007fae55334000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-27" daemon [_thread_blocked, id=60679, stack(0x0000700015705000,0x0000700015805000)]
  0x00007fae54f1f800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-26" daemon [_thread_blocked, id=60467, stack(0x0000700015602000,0x0000700015702000)]
  0x00007fae5631e000 JavaThread "inner-job-com.ejlerp.saleorder.jobs.job.OutOfStockToNormalFromStockInventoryJob-4" daemon [_thread_blocked, id=60163, stack(0x00007000154ff000,0x00007000155ff000)]
  0x00007fae55cc1800 JavaThread "inner-job-com.ejlerp.saleorder.jobs.job.OutOfStockToNormalFromStockInventoryJob-3" daemon [_thread_blocked, id=69635, stack(0x00007000153fc000,0x00007000154fc000)]
  0x00007fae54d20000 JavaThread "job-event-8" daemon [_thread_blocked, id=70163, stack(0x00007000152f9000,0x00007000153f9000)]
  0x00007fae5631f800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-25" daemon [_thread_blocked, id=59683, stack(0x00007000151f6000,0x00007000152f6000)]
  0x00007fae55383800 JavaThread "job-event-7" daemon [_thread_blocked, id=59399, stack(0x00007000150f3000,0x00007000151f3000)]
  0x00007fae56316800 JavaThread "job-event-6" daemon [_thread_blocked, id=58883, stack(0x0000700014ff0000,0x00007000150f0000)]
  0x00007fae54d06800 JavaThread "job-event-5" daemon [_thread_blocked, id=58379, stack(0x0000700014eed000,0x0000700014fed000)]
  0x00007fae56316000 JavaThread "DubboResponseTimeoutScanTimer" daemon [_thread_blocked, id=70915, stack(0x0000700014dea000,0x0000700014eea000)]
  0x00007fae56409800 JavaThread "job-event-3" daemon [_thread_blocked, id=71171, stack(0x0000700014ce7000,0x0000700014de7000)]
  0x00007fae5594e800 JavaThread "job-event-4" daemon [_thread_blocked, id=57859, stack(0x0000700014be4000,0x0000700014ce4000)]
  0x00007fae5594d800 JavaThread "inner-job-com.ejlerp.saleorder.jobs.job.OutOfStockToNormalFromStockInventoryJob-2" daemon [_thread_blocked, id=71683, stack(0x0000700014ae1000,0x0000700014be1000)]
  0x00007fae56406800 JavaThread "inner-job-com.ejlerp.saleorder.jobs.job.OutOfStockToNormalFromStockInventoryJob-1" daemon [_thread_blocked, id=71939, stack(0x00007000149de000,0x0000700014ade000)]
  0x00007fae54cfa800 JavaThread "job-event-2" daemon [_thread_blocked, id=72195, stack(0x00007000148db000,0x00007000149db000)]
  0x00007fae56405000 JavaThread "job-event-1" daemon [_thread_blocked, id=72723, stack(0x00007000147d8000,0x00007000148d8000)]
  0x00007fae54d48800 JavaThread "RMI TCP Connection(5)-192.168.0.123" daemon [_thread_in_native, id=73223, stack(0x00007000146d5000,0x00007000147d5000)]
  0x00007fae55341000 JavaThread "ReconcileService" [_thread_blocked, id=73739, stack(0x00007000145d2000,0x00007000146d2000)]
  0x00007fae55362800 JavaThread "Curator-TreeCache-6" daemon [_thread_blocked, id=56839, stack(0x00007000144cf000,0x00007000145cf000)]
  0x00007fae55361800 JavaThread "Timer-6" daemon [_thread_blocked, id=56323, stack(0x00007000143cc000,0x00007000144cc000)]
  0x00007fae5644d800 JavaThread "com.ejlerp.saleorder.jobs.job.sms.WaitingForEvalRemindJob_QuartzSchedulerThread" [_thread_blocked, id=56067, stack(0x00007000142c9000,0x00007000143c9000)]
  0x00007fae5644d000 JavaThread "com.ejlerp.saleorder.jobs.job.sms.WaitingForEvalRemindJob_Worker-1" [_thread_blocked, id=55555, stack(0x00007000141c6000,0x00007000142c6000)]
  0x00007fae55361000 JavaThread "ReconcileService" [_thread_blocked, id=55311, stack(0x00007000140c3000,0x00007000141c3000)]
  0x00007fae552de000 JavaThread "Curator-TreeCache-5" daemon [_thread_blocked, id=55047, stack(0x0000700013fc0000,0x00007000140c0000)]
  0x00007fae54ca0800 JavaThread "Timer-5" daemon [_thread_blocked, id=54531, stack(0x0000700013ebd000,0x0000700013fbd000)]
  0x00007fae54ca0000 JavaThread "com.ejlerp.saleorder.jobs.job.sms.AfterSaleMessageJob_QuartzSchedulerThread" [_thread_blocked, id=75011, stack(0x0000700013dba000,0x0000700013eba000)]
  0x00007fae552dd000 JavaThread "com.ejlerp.saleorder.jobs.job.sms.AfterSaleMessageJob_Worker-1" [_thread_blocked, id=54019, stack(0x0000700013cb7000,0x0000700013db7000)]
  0x00007fae5534b800 JavaThread "ReconcileService" [_thread_blocked, id=75779, stack(0x0000700013bb4000,0x0000700013cb4000)]
  0x00007fae55a1c000 JavaThread "Curator-TreeCache-4" daemon [_thread_blocked, id=76295, stack(0x0000700013ab1000,0x0000700013bb1000)]
  0x00007fae5534b000 JavaThread "Timer-4" daemon [_thread_blocked, id=76547, stack(0x00007000139ae000,0x0000700013aae000)]
  0x00007fae55a3b000 JavaThread "com.ejlerp.saleorder.jobs.job.sms.WaitingForReceivePaymentJob_QuartzSchedulerThread" [_thread_blocked, id=53507, stack(0x00007000138ab000,0x00007000139ab000)]
  0x00007fae54c96000 JavaThread "com.ejlerp.saleorder.jobs.job.sms.WaitingForReceivePaymentJob_Worker-1" [_thread_blocked, id=52999, stack(0x00007000137a8000,0x00007000138a8000)]
  0x00007fae5644a000 JavaThread "ReconcileService" [_thread_blocked, id=52743, stack(0x00007000136a5000,0x00007000137a5000)]
  0x00007fae56449000 JavaThread "Curator-TreeCache-3" daemon [_thread_blocked, id=77323, stack(0x00007000135a2000,0x00007000136a2000)]
  0x00007fae54c95800 JavaThread "Timer-3" daemon [_thread_blocked, id=52227, stack(0x000070001349f000,0x000070001359f000)]
  0x00007fae54c94800 JavaThread "com.ejlerp.saleorder.jobs.job.sms.ShipmentRemindJob_QuartzSchedulerThread" [_thread_blocked, id=51971, stack(0x000070001339c000,0x000070001349c000)]
  0x00007fae54c94000 JavaThread "com.ejlerp.saleorder.jobs.job.sms.ShipmentRemindJob_Worker-1" [_thread_blocked, id=51459, stack(0x0000700013299000,0x0000700013399000)]
  0x00007fae564e6800 JavaThread "ReconcileService" [_thread_blocked, id=78351, stack(0x0000700013196000,0x0000700013296000)]
  0x00007fae54c92800 JavaThread "Curator-TreeCache-2" daemon [_thread_blocked, id=51207, stack(0x0000700013093000,0x0000700013193000)]
  0x00007fae562ef800 JavaThread "Timer-2" daemon [_thread_blocked, id=79107, stack(0x0000700012f90000,0x0000700013090000)]
  0x00007fae562ef000 JavaThread "com.ejlerp.saleorder.jobs.job.sms.PushPaymentJob_QuartzSchedulerThread" [_thread_blocked, id=79363, stack(0x0000700012e8d000,0x0000700012f8d000)]
  0x00007fae54c91800 JavaThread "com.ejlerp.saleorder.jobs.job.sms.PushPaymentJob_Worker-1" [_thread_blocked, id=79895, stack(0x0000700012d8a000,0x0000700012e8a000)]
  0x00007fae5533d000 JavaThread "ReconcileService" [_thread_blocked, id=80131, stack(0x0000700012c87000,0x0000700012d87000)]
  0x00007fae552a6800 JavaThread "Curator-TreeCache-1" daemon [_thread_blocked, id=50187, stack(0x0000700012b84000,0x0000700012c84000)]
  0x00007fae55a3d800 JavaThread "Timer-1" daemon [_thread_blocked, id=49667, stack(0x0000700012a81000,0x0000700012b81000)]
  0x00007fae55a3c800 JavaThread "com.ejlerp.saleorder.jobs.job.OutOfStockToNormalFromStockInventoryJob_QuartzSchedulerThread" [_thread_blocked, id=49411, stack(0x000070001297e000,0x0000700012a7e000)]
  0x00007fae55be1000 JavaThread "com.ejlerp.saleorder.jobs.job.OutOfStockToNormalFromStockInventoryJob_Worker-1" [_thread_blocked, id=49155, stack(0x000070001287b000,0x000070001297b000)]
  0x00007fae55be0800 JavaThread "ReconcileService" [_thread_blocked, id=80915, stack(0x0000700012778000,0x0000700012878000)]
  0x00007fae56400000 JavaThread "Curator-TreeCache-0" daemon [_thread_blocked, id=81411, stack(0x0000700012675000,0x0000700012775000)]
  0x00007fae563ff800 JavaThread "Timer-0" daemon [_thread_blocked, id=48643, stack(0x0000700012572000,0x0000700012672000)]
  0x00007fae56112800 JavaThread "com.ejlerp.saleorder.jobs.job.AutoCheckOrderJob_QuartzSchedulerThread" [_thread_blocked, id=48387, stack(0x000070001246f000,0x000070001256f000)]
  0x00007fae562bf800 JavaThread "com.ejlerp.saleorder.jobs.job.AutoCheckOrderJob_Worker-1" [_thread_blocked, id=82439, stack(0x000070001236c000,0x000070001246c000)]
  0x00007fae54ae4800 JavaThread "DubboClientHandler-192.168.0.157:20881-thread-1" daemon [_thread_blocked, id=82703, stack(0x0000700012269000,0x0000700012369000)]
  0x00007fae5533e800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-24" daemon [_thread_blocked, id=82947, stack(0x0000700012166000,0x0000700012266000)]
  0x00007fae54a64800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-23" daemon [_thread_blocked, id=47363, stack(0x0000700012063000,0x0000700012163000)]
  0x00007fae560f3000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-22" daemon [_thread_blocked, id=83459, stack(0x0000700011f60000,0x0000700012060000)]
  0x00007fae54b1b800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-21" daemon [_thread_blocked, id=83971, stack(0x0000700011e5d000,0x0000700011f5d000)]
  0x00007fae54b1a800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-20" daemon [_thread_blocked, id=84227, stack(0x0000700011d5a000,0x0000700011e5a000)]
  0x00007fae563f3000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-19" daemon [_thread_blocked, id=46595, stack(0x0000700011c57000,0x0000700011d57000)]
  0x00007fae54b1a000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-18" daemon [_thread_blocked, id=46083, stack(0x0000700011b54000,0x0000700011c54000)]
  0x00007fae54b7f000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-17" daemon [_thread_blocked, id=45831, stack(0x000070001194e000,0x0000700011a4e000)]
  0x00007fae563f2000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-16" daemon [_thread_blocked, id=84739, stack(0x0000700011a51000,0x0000700011b51000)]
  0x00007fae54b7e000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-15" daemon [_thread_blocked, id=45571, stack(0x000070001184b000,0x000070001194b000)]
  0x00007fae54a3f800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-14" daemon [_thread_blocked, id=85507, stack(0x0000700011748000,0x0000700011848000)]
  0x00007fae562a7000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-13" daemon [_thread_blocked, id=45059, stack(0x0000700011645000,0x0000700011745000)]
  0x00007fae54a54800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-12" daemon [_thread_blocked, id=86275, stack(0x0000700011542000,0x0000700011642000)]
  0x00007fae54a54000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-11" daemon [_thread_blocked, id=44547, stack(0x000070001143f000,0x000070001153f000)]
  0x00007fae54b76000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-10" daemon [_thread_blocked, id=44035, stack(0x000070001133c000,0x000070001143c000)]
  0x00007fae551ee800 JavaThread "Druid-ConnectionPool-Destroy-2073021938" daemon [_thread_blocked, id=86787, stack(0x0000700011239000,0x0000700011339000)]
  0x00007fae55202800 JavaThread "Druid-ConnectionPool-Create-2073021938" daemon [_thread_blocked, id=87051, stack(0x0000700011136000,0x0000700011236000)]
  0x00007fae564b8000 JavaThread "Abandoned connection cleanup thread" daemon [_thread_blocked, id=32003, stack(0x0000700011033000,0x0000700011133000)]
  0x00007fae54976000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-9" daemon [_thread_blocked, id=31763, stack(0x0000700010f30000,0x0000700011030000)]
  0x00007fae552c4000 JavaThread "DubboClientHandler-192.168.0.134:20880-thread-1" daemon [_thread_blocked, id=31495, stack(0x0000700010e2d000,0x0000700010f2d000)]
  0x00007fae54b36800 JavaThread "DubboClientHandler-192.168.0.157:20887-thread-1" daemon [_thread_blocked, id=31003, stack(0x0000700010d2a000,0x0000700010e2a000)]
  0x00007fae5622d000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-8" daemon [_thread_blocked, id=30723, stack(0x0000700010c27000,0x0000700010d27000)]
  0x00007fae54b91000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-7" daemon [_thread_blocked, id=33031, stack(0x0000700010b24000,0x0000700010c24000)]
  0x00007fae54b8b000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-6" daemon [_thread_blocked, id=33283, stack(0x0000700010a21000,0x0000700010b21000)]
  0x00007fae56231800 JavaThread "DubboClientHandler-192.168.0.157:20883-thread-1" daemon [_thread_blocked, id=29955, stack(0x000070001091e000,0x0000700010a1e000)]
  0x00007fae559ae000 JavaThread "dubbo-remoting-client-heartbeat-thread-2" daemon [_thread_blocked, id=29443, stack(0x000070001081b000,0x000070001091b000)]
  0x00007fae552a3800 JavaThread "DubboClientHandler-192.168.0.126:20882-thread-1" daemon [_thread_blocked, id=34067, stack(0x0000700010718000,0x0000700010818000)]
  0x00007fae56228000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-5" daemon [_thread_blocked, id=28939, stack(0x0000700010615000,0x0000700010715000)]
  0x00007fae56200000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-4" daemon [_thread_blocked, id=34307, stack(0x0000700010512000,0x0000700010612000)]
  0x00007fae54bbf800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-3" daemon [_thread_blocked, id=28431, stack(0x000070001040f000,0x000070001050f000)]
  0x00007fae56421800 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-2" daemon [_thread_blocked, id=28163, stack(0x000070001030c000,0x000070001040c000)]
  0x00007fae54b66000 JavaThread "DubboServerHandler-192.168.0.123:20881-thread-1" daemon [_thread_blocked, id=35339, stack(0x0000700010209000,0x0000700010309000)]
  0x00007fae54bf4800 JavaThread "dubbo-remoting-server-heartbeat-thread-1" daemon [_thread_blocked, id=27651, stack(0x0000700010106000,0x0000700010206000)]
  0x00007fae54bf3800 JavaThread "New I/O server boss #12" daemon [_thread_in_native, id=27139, stack(0x0000700010003000,0x0000700010103000)]
  0x00007fae54b63000 JavaThread "New I/O worker #11" daemon [_thread_in_native, id=35843, stack(0x000070000ff00000,0x0000700010000000)]
  0x00007fae54bf3000 JavaThread "New I/O worker #10" daemon [_thread_in_native, id=36099, stack(0x000070000fdfd000,0x000070000fefd000)]
  0x00007fae54bf2000 JavaThread "New I/O worker #9" daemon [_thread_in_native, id=26371, stack(0x000070000fcfa000,0x000070000fdfa000)]
  0x00007fae56421000 JavaThread "New I/O worker #8" daemon [_thread_in_native, id=36611, stack(0x000070000fbf7000,0x000070000fcf7000)]
  0x00007fae54bf1800 JavaThread "New I/O worker #7" daemon [_thread_in_native, id=37139, stack(0x000070000faf4000,0x000070000fbf4000)]
  0x00007fae54c5f800 JavaThread "DubboClientReconnectTimer-thread-2" daemon [_thread_blocked, id=26131, stack(0x000070000f9f1000,0x000070000faf1000)]
  0x00007fae563e4800 JavaThread "dubbo-remoting-client-heartbeat-thread-1" daemon [_thread_blocked, id=37891, stack(0x000070000f8ee000,0x000070000f9ee000)]
  0x00007fae563e4000 JavaThread "DubboClientHandler-192.168.0.157:20880-thread-1" daemon [_thread_blocked, id=25859, stack(0x000070000f7eb000,0x000070000f8eb000)]
  0x00007fae54c3c800 JavaThread "Hashed wheel timer #1" [_thread_blocked, id=25603, stack(0x000070000f6e8000,0x000070000f7e8000)]
  0x00007fae56016800 JavaThread "DubboClientReconnectTimer-thread-1" daemon [_thread_blocked, id=25091, stack(0x000070000f5e5000,0x000070000f6e5000)]
  0x00007fae54a33800 JavaThread "New I/O boss #6" daemon [_thread_in_native, id=38659, stack(0x000070000f4e2000,0x000070000f5e2000)]
  0x00007fae54c3e000 JavaThread "New I/O worker #5" daemon [_thread_blocked, id=24579, stack(0x000070000f3df000,0x000070000f4df000)]
  0x00007fae561d4800 JavaThread "New I/O worker #4" daemon [_thread_in_native, id=39427, stack(0x000070000f2dc000,0x000070000f3dc000)]
  0x00007fae561c7800 JavaThread "New I/O worker #3" daemon [_thread_in_native, id=39683, stack(0x000070000f1d9000,0x000070000f2d9000)]
  0x00007fae561c6800 JavaThread "New I/O worker #2" daemon [_thread_in_native, id=24067, stack(0x000070000f0d6000,0x000070000f1d6000)]
  0x00007fae561cb800 JavaThread "New I/O worker #1" daemon [_thread_blocked, id=23811, stack(0x000070000efd3000,0x000070000f0d3000)]
  0x00007fae55c61000 JavaThread "DubboSaveRegistryCache-thread-1" daemon [_thread_blocked, id=40459, stack(0x000070000eed0000,0x000070000efd0000)]
  0x00007fae561c6000 JavaThread "main-EventThread" daemon [_thread_blocked, id=40963, stack(0x000070000edcd000,0x000070000eecd000)]
  0x00007fae56502000 JavaThread "main-SendThread(192.168.0.156:2181)" daemon [_thread_in_native, id=41475, stack(0x000070000ecca000,0x000070000edca000)]
  0x00007fae56501000 JavaThread "ZkClient-EventThread-50-192.168.0.156:2181" daemon [_thread_blocked, id=23307, stack(0x000070000ebc7000,0x000070000ecc7000)]
  0x00007fae564fc000 JavaThread "DubboRegistryFailedRetryTimer-thread-1" daemon [_thread_blocked, id=5703, stack(0x000070000eac4000,0x000070000ebc4000)]
  0x00007fae55283800 JavaThread "Curator-Framework-0" daemon [_thread_blocked, id=42243, stack(0x000070000e9c1000,0x000070000eac1000)]
  0x00007fae55282800 JavaThread "main-EventThread" daemon [_thread_blocked, id=42499, stack(0x000070000e8be000,0x000070000e9be000)]
  0x00007fae564f2800 JavaThread "main-SendThread(192.168.0.156:2181)" daemon [_thread_blocked, id=22787, stack(0x000070000e7bb000,0x000070000e8bb000)]
  0x00007fae5526d800 JavaThread "Curator-ConnectionStateManager-0" daemon [_thread_blocked, id=43047, stack(0x000070000e6b8000,0x000070000e7b8000)]
  0x00007fae55aad000 JavaThread "RMI TCP Connection(3)-192.168.0.123" daemon [_thread_in_native, id=43267, stack(0x000070000e5b5000,0x000070000e6b5000)]
  0x00007fae551e2000 JavaThread "RMI Scheduler(0)" daemon [_thread_blocked, id=21763, stack(0x000070000e4b2000,0x000070000e5b2000)]
  0x00007fae5497b000 JavaThread "RMI TCP Accept-0" daemon [_thread_in_native, id=15875, stack(0x000070000e2ac000,0x000070000e3ac000)]
  0x00007fae55a09000 JavaThread "RMI TCP Connection(1)-127.0.0.1" daemon [_thread_in_native, id=15363, stack(0x000070000e1a9000,0x000070000e2a9000)]
  0x00007fae55a16000 JavaThread "RMI TCP Accept-52007" daemon [_thread_in_native, id=16899, stack(0x000070000e0a6000,0x000070000e1a6000)]
  0x00007fae5617f000 JavaThread "RMI TCP Accept-0" daemon [_thread_in_native, id=17155, stack(0x000070000dfa3000,0x000070000e0a3000)]
  0x00007fae54956800 JavaThread "Service Thread" daemon [_thread_blocked, id=17411, stack(0x000070000dea0000,0x000070000dfa0000)]
  0x00007fae560f0000 JavaThread "C1 CompilerThread2" daemon [_thread_blocked, id=14083, stack(0x000070000dd9d000,0x000070000de9d000)]
  0x00007fae5593f000 JavaThread "C2 CompilerThread1" daemon [_thread_blocked, id=13571, stack(0x000070000dc9a000,0x000070000dd9a000)]
  0x00007fae560ef800 JavaThread "C2 CompilerThread0" daemon [_thread_blocked, id=17923, stack(0x000070000db97000,0x000070000dc97000)]
  0x00007fae55034000 JavaThread "Monitor Ctrl-Break" daemon [_thread_in_native, id=18435, stack(0x000070000da94000,0x000070000db94000)]
  0x00007fae5600f000 JavaThread "Signal Dispatcher" daemon [_thread_blocked, id=12811, stack(0x000070000d991000,0x000070000da91000)]
  0x00007fae5482b000 JavaThread "Finalizer" daemon [_thread_blocked, id=11779, stack(0x000070000d88e000,0x000070000d98e000)]
  0x00007fae55032800 JavaThread "Reference Handler" daemon [_thread_blocked, id=20995, stack(0x000070000d78b000,0x000070000d88b000)]

Other Threads:
=>0x00007fae5502f800 VMThread [stack: 0x000070000d688000,0x000070000d788000] [id=21251]
  0x00007fae5497b800 WatcherThread [stack: 0x000070000e3af000,0x000070000e4af000] [id=16131]

VM state:at safepoint (normal execution)

VM Mutex/Monitor currently owned by a thread:  ([mutex/lock_event])
[0x00007fae54601800] Threads_lock - owner thread: 0x00007fae5502f800
[0x00007fae54601d00] Heap_lock - owner thread: 0x00007fae55a24000

Heap:
 PSYoungGen      total 564736K, used 19670K [0x0000000795580000, 0x00000007bf780000, 0x00000007c0000000)
  eden space 544768K, 0% used [0x0000000795580000,0x0000000795580000,0x00000007b6980000)
  from space 19968K, 98% used [0x00000007b6980000,0x00000007b7cb5a10,0x00000007b7d00000)
  to   space 72704K, 0% used [0x00000007bb080000,0x00000007bb080000,0x00000007bf780000)
 ParOldGen       total 191488K, used 137264K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x000000074860c0d0,0x000000074bb00000)
 Metaspace       used 43836K, capacity 45586K, committed 45656K, reserved 1089536K
  class space    used 5324K, capacity 5612K, committed 5632K, reserved 1048576K

Card table byte_map: [0x0000000102bc0000,0x0000000102fc1000] byte_map_base: 0x00000000ff1c0000

Marking Bits: (ParMarkBitMap*) 0x000000010182d5d0
 Begin Bits: [0x000000010326c000, 0x000000010526c000)
 End Bits:   [0x000000010526c000, 0x000000010726c000)

Polling page: 0x0000000101f24000

CodeCache: size=245760Kb used=13228Kb max_used=13248Kb free=232531Kb
 bounds [0x000000010d78b000, 0x000000010e48b000, 0x000000011c78b000]
 total_blobs=6800 nmethods=6274 adapters=438
 compilation: enabled

Compilation events (10 events):
Event: 72.788 Thread 0x00007fae560f0000 nmethod 6375 0x000000010dd32b10 code [0x000000010dd32c80, 0x000000010dd32da8]
Event: 72.790 Thread 0x00007fae560f0000 6376       1       com.sun.jmx.mbeanserver.MBeanSupport::invoke (19 bytes)
Event: 72.790 Thread 0x00007fae560f0000 nmethod 6376 0x000000010dd50590 code [0x000000010dd50700, 0x000000010dd50898]
Event: 72.792 Thread 0x00007fae560f0000 6377       1       javax.management.remote.rmi.RMIConnectionImpl::nullIsEmpty (12 bytes)
Event: 72.793 Thread 0x00007fae560f0000 nmethod 6377 0x000000010dd502d0 code [0x000000010dd50420, 0x000000010dd50530]
Event: 72.793 Thread 0x00007fae560f0000 6378       1       com.sun.jmx.mbeanserver.PerInterface::invoke (269 bytes)
Event: 72.797 Thread 0x00007fae560f0000 nmethod 6378 0x000000010dd3e510 code [0x000000010dd3e820, 0x000000010dd3f4a8]
Event: 72.797 Thread 0x00007fae560f0000 6379       1       javax.management.remote.rmi.RMIConnectionImpl$6::run (17 bytes)
Event: 72.797 Thread 0x00007fae560f0000 nmethod 6379 0x000000010dd3e190 code [0x000000010dd3e300, 0x000000010dd3e448]
Event: 72.797 Thread 0x00007fae560f0000 6380   !   1       javax.management.remote.rmi.RMIConnectionImpl::unwrap (301 bytes)

GC Heap History (10 events):
Event: 54.010 GC heap after
Heap after GC invocations=24 (full 3):
 PSYoungGen      total 611840K, used 34359K [0x0000000795580000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 534528K, 0% used [0x0000000795580000,0x0000000795580000,0x00000007b5f80000)
  from space 77312K, 44% used [0x00000007b5f80000,0x00000007b810df98,0x00000007bab00000)
  to   space 75264K, 0% used [0x00000007bb680000,0x00000007bb680000,0x00000007c0000000)
 ParOldGen       total 191488K, used 137232K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x00000007486040d0,0x000000074bb00000)
 Metaspace       used 40321K, capacity 41992K, committed 42112K, reserved 1085440K
  class space    used 4923K, capacity 5182K, committed 5248K, reserved 1048576K
}
Event: 55.308 GC heap before
{Heap before GC invocations=25 (full 3):
 PSYoungGen      total 611840K, used 568887K [0x0000000795580000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 534528K, 100% used [0x0000000795580000,0x00000007b5f80000,0x00000007b5f80000)
  from space 77312K, 44% used [0x00000007b5f80000,0x00000007b810df98,0x00000007bab00000)
  to   space 75264K, 0% used [0x00000007bb680000,0x00000007bb680000,0x00000007c0000000)
 ParOldGen       total 191488K, used 137232K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x00000007486040d0,0x000000074bb00000)
 Metaspace       used 40816K, capacity 42512K, committed 42624K, reserved 1087488K
  class space    used 5004K, capacity 5248K, committed 5248K, reserved 1048576K
Event: 55.332 GC heap after
Heap after GC invocations=25 (full 3):
 PSYoungGen      total 617472K, used 47568K [0x0000000795580000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 542208K, 0% used [0x0000000795580000,0x0000000795580000,0x00000007b6700000)
  from space 75264K, 63% used [0x00000007bb680000,0x00000007be4f4270,0x00000007c0000000)
  to   space 78336K, 0% used [0x00000007b6700000,0x00000007b6700000,0x00000007bb380000)
 ParOldGen       total 191488K, used 137240K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x00000007486060d0,0x000000074bb00000)
 Metaspace       used 40816K, capacity 42512K, committed 42624K, reserved 1087488K
  class space    used 5004K, capacity 5248K, committed 5248K, reserved 1048576K
}
Event: 56.843 GC heap before
{Heap before GC invocations=26 (full 3):
 PSYoungGen      total 617472K, used 589776K [0x0000000795580000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 542208K, 100% used [0x0000000795580000,0x00000007b6700000,0x00000007b6700000)
  from space 75264K, 63% used [0x00000007bb680000,0x00000007be4f4270,0x00000007c0000000)
  to   space 78336K, 0% used [0x00000007b6700000,0x00000007b6700000,0x00000007bb380000)
 ParOldGen       total 191488K, used 137240K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x00000007486060d0,0x000000074bb00000)
 Metaspace       used 42398K, capacity 44102K, committed 44416K, reserved 1087488K
  class space    used 5172K, capacity 5449K, committed 5504K, reserved 1048576K
Event: 56.881 GC heap after
Heap after GC invocations=26 (full 3):
 PSYoungGen      total 586752K, used 44259K [0x0000000795580000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 542208K, 0% used [0x0000000795580000,0x0000000795580000,0x00000007b6700000)
  from space 44544K, 99% used [0x00000007b6700000,0x00000007b9238de0,0x00000007b9280000)
  to   space 78848K, 0% used [0x00000007bb300000,0x00000007bb300000,0x00000007c0000000)
 ParOldGen       total 191488K, used 137248K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x00000007486080d0,0x000000074bb00000)
 Metaspace       used 42398K, capacity 44102K, committed 44416K, reserved 1087488K
  class space    used 5172K, capacity 5449K, committed 5504K, reserved 1048576K
}
Event: 71.208 GC heap before
{Heap before GC invocations=27 (full 3):
 PSYoungGen      total 586752K, used 586467K [0x0000000795580000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 542208K, 100% used [0x0000000795580000,0x00000007b6700000,0x00000007b6700000)
  from space 44544K, 99% used [0x00000007b6700000,0x00000007b9238de0,0x00000007b9280000)
  to   space 78848K, 0% used [0x00000007bb300000,0x00000007bb300000,0x00000007c0000000)
 ParOldGen       total 191488K, used 137248K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x00000007486080d0,0x000000074bb00000)
 Metaspace       used 43443K, capacity 45208K, committed 45312K, reserved 1089536K
  class space    used 5264K, capacity 5565K, committed 5632K, reserved 1048576K
Event: 71.246 GC heap after
Heap after GC invocations=27 (full 3):
 PSYoungGen      total 623616K, used 33938K [0x0000000795580000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 544768K, 0% used [0x0000000795580000,0x0000000795580000,0x00000007b6980000)
  from space 78848K, 43% used [0x00000007bb300000,0x00000007bd424850,0x00000007c0000000)
  to   space 75264K, 0% used [0x00000007b6980000,0x00000007b6980000,0x00000007bb300000)
 ParOldGen       total 191488K, used 137256K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x000000074860a0d0,0x000000074bb00000)
 Metaspace       used 43443K, capacity 45208K, committed 45312K, reserved 1089536K
  class space    used 5264K, capacity 5565K, committed 5632K, reserved 1048576K
}
Event: 72.798 GC heap before
{Heap before GC invocations=28 (full 3):
 PSYoungGen      total 623616K, used 66204K [0x0000000795580000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 544768K, 5% used [0x0000000795580000,0x0000000797502a20,0x00000007b6980000)
  from space 78848K, 43% used [0x00000007bb300000,0x00000007bd424850,0x00000007c0000000)
  to   space 75264K, 0% used [0x00000007b6980000,0x00000007b6980000,0x00000007bb300000)
 ParOldGen       total 191488K, used 137256K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x000000074860a0d0,0x000000074bb00000)
 Metaspace       used 43836K, capacity 45586K, committed 45656K, reserved 1089536K
  class space    used 5324K, capacity 5612K, committed 5632K, reserved 1048576K
Event: 72.833 GC heap after
Heap after GC invocations=28 (full 3):
 PSYoungGen      total 564736K, used 19670K [0x0000000795580000, 0x00000007bf780000, 0x00000007c0000000)
  eden space 544768K, 0% used [0x0000000795580000,0x0000000795580000,0x00000007b6980000)
  from space 19968K, 98% used [0x00000007b6980000,0x00000007b7cb5a10,0x00000007b7d00000)
  to   space 72704K, 0% used [0x00000007bb080000,0x00000007bb080000,0x00000007bf780000)
 ParOldGen       total 191488K, used 137264K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x000000074860c0d0,0x000000074bb00000)
 Metaspace       used 43836K, capacity 45586K, committed 45656K, reserved 1089536K
  class space    used 5324K, capacity 5612K, committed 5632K, reserved 1048576K
}
Event: 72.833 GC heap before
{Heap before GC invocations=29 (full 4):
 PSYoungGen      total 564736K, used 19670K [0x0000000795580000, 0x00000007bf780000, 0x00000007c0000000)
  eden space 544768K, 0% used [0x0000000795580000,0x0000000795580000,0x00000007b6980000)
  from space 19968K, 98% used [0x00000007b6980000,0x00000007b7cb5a10,0x00000007b7d00000)
  to   space 72704K, 0% used [0x00000007bb080000,0x00000007bb080000,0x00000007bf780000)
 ParOldGen       total 191488K, used 137264K [0x0000000740000000, 0x000000074bb00000, 0x0000000795580000)
  object space 191488K, 71% used [0x0000000740000000,0x000000074860c0d0,0x000000074bb00000)
 Metaspace       used 43836K, capacity 45586K, committed 45656K, reserved 1089536K
  class space    used 5324K, capacity 5612K, committed 5632K, reserved 1048576K

Deoptimization events (0 events):
No events

Internal exceptions (10 events):
Event: 71.281 Thread 0x00007fae55808800 Exception <a 'java/lang/ClassNotFoundException': org/springframework/jmx/export/metadata/ManagedAttributeBeanInfo> (0x0000000795895208) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/classfile/systemDictiona
Event: 71.281 Thread 0x00007fae55808800 Exception <a 'java/lang/ClassNotFoundException': org/springframework/jmx/export/metadata/ManagedAttributeCustomizer> (0x00000007958b3560) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/classfile/systemDictio
Event: 71.290 Thread 0x00007fae55808800 Exception <a 'java/lang/ClassNotFoundException': org/springframework/boot/actuate/endpoint/jmx/JmxEndpointCustomizer> (0x0000000795901e88) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/classfile/systemDicti
Event: 71.307 Thread 0x00007fae55808800 Exception <a 'java/lang/ClassNotFoundException': org/springframework/boot/actuate/endpoint/jmx/LoggersEndpointMBeanBeanInfo> (0x00000007959c81d8) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/classfile/syst
Event: 71.307 Thread 0x00007fae55808800 Exception <a 'java/lang/ClassNotFoundException': org/springframework/boot/actuate/endpoint/jmx/LoggersEndpointMBeanCustomizer> (0x00000007959ea158) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/classfile/sy
Event: 71.309 Thread 0x00007fae55808800 Exception <a 'java/lang/ClassNotFoundException': org/springframework/boot/actuate/endpoint/jmx/JmxEndpointCustomizer> (0x0000000795a1a220) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/classfile/systemDicti
Event: 71.327 Thread 0x00007fae55808800 Exception <a 'java/lang/ArrayIndexOutOfBoundsException'> (0x0000000795b18538) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/runtime/sharedRuntime.cpp, line 605]
Event: 71.329 Thread 0x00007fae55808800 Exception <a 'java/lang/ArrayIndexOutOfBoundsException'> (0x0000000795b2dd00) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/runtime/sharedRuntime.cpp, line 605]
Event: 71.329 Thread 0x00007fae55808800 Exception <a 'java/lang/ArrayIndexOutOfBoundsException'> (0x0000000795b2ead8) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/runtime/sharedRuntime.cpp, line 605]
Event: 71.329 Thread 0x00007fae55808800 Exception <a 'java/lang/ArrayIndexOutOfBoundsException'> (0x0000000795b34770) thrown at [/Users/java_re/workspace/8-2-build-macosx-x86_64/jdk8u151/9699/hotspot/src/share/vm/runtime/sharedRuntime.cpp, line 605]

Events (10 events):
Event: 72.768 Thread 0x00007fae560f0000 flushing nmethod 0x000000010df51bd0
Event: 72.789 loading class org/springframework/boot/actuate/health/OrderedHealthAggregator$StatusComparator
Event: 72.789 loading class org/springframework/boot/actuate/health/OrderedHealthAggregator$StatusComparator done
Event: 72.791 loading class com/fasterxml/jackson/core/json/JsonWriteContext
Event: 72.791 loading class com/fasterxml/jackson/core/json/JsonWriteContext done
Event: 72.792 loading class com/fasterxml/jackson/core/JsonStreamContext
Event: 72.792 loading class com/fasterxml/jackson/core/JsonStreamContext done
Event: 72.797 loading class com/fasterxml/jackson/databind/util/TokenBuffer$Segment
Event: 72.797 loading class com/fasterxml/jackson/databind/util/TokenBuffer$Segment done
Event: 72.798 Executing VM operation: CollectForMetadataAllocation


Dynamic libraries:
0x000000001832f000  /System/Library/Frameworks/Cocoa.framework/Versions/A/Cocoa
0x000000001832f000  /System/Library/Frameworks/Security.framework/Versions/A/Security
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/ApplicationServices
0x000000001832f000  /usr/lib/libz.1.dylib
0x000000001832f000  /usr/lib/libSystem.B.dylib
0x000000001832f000  /usr/lib/libobjc.A.dylib
0x000000001832f000  /System/Library/Frameworks/CoreFoundation.framework/Versions/A/CoreFoundation
0x000000001832f000  /System/Library/Frameworks/Foundation.framework/Versions/C/Foundation
0x000000001832f000  /System/Library/Frameworks/AppKit.framework/Versions/C/AppKit
0x000000001832f000  /System/Library/Frameworks/CoreData.framework/Versions/A/CoreData
0x000000001832f000  /System/Library/PrivateFrameworks/RemoteViewServices.framework/Versions/A/RemoteViewServices
0x000000001832f000  /System/Library/PrivateFrameworks/UIFoundation.framework/Versions/A/UIFoundation
0x000000001832f000  /System/Library/PrivateFrameworks/DFRFoundation.framework/Versions/A/DFRFoundation
0x000000001832f000  /System/Library/Frameworks/Metal.framework/Versions/A/Metal
0x000000001832f000  /System/Library/PrivateFrameworks/DesktopServicesPriv.framework/Versions/A/DesktopServicesPriv
0x000000001832f000  /usr/lib/libenergytrace.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/SkyLight.framework/Versions/A/SkyLight
0x000000001832f000  /System/Library/Frameworks/CoreGraphics.framework/Versions/A/CoreGraphics
0x000000001832f000  /usr/lib/libScreenReader.dylib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Accelerate
0x000000001832f000  /System/Library/Frameworks/IOSurface.framework/Versions/A/IOSurface
0x000000001832f000  /System/Library/Frameworks/AudioToolbox.framework/Versions/A/AudioToolbox
0x000000001832f000  /System/Library/Frameworks/AudioUnit.framework/Versions/A/AudioUnit
0x000000001832f000  /System/Library/PrivateFrameworks/DataDetectorsCore.framework/Versions/A/DataDetectorsCore
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/HIToolbox.framework/Versions/A/HIToolbox
0x000000001832f000  /usr/lib/libicucore.A.dylib
0x000000001832f000  /System/Library/Frameworks/QuartzCore.framework/Versions/A/QuartzCore
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/SpeechRecognition.framework/Versions/A/SpeechRecognition
0x000000001832f000  /usr/lib/libauto.dylib
0x000000001832f000  /usr/lib/libxml2.2.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/CoreUI.framework/Versions/A/CoreUI
0x000000001832f000  /System/Library/Frameworks/CoreAudio.framework/Versions/A/CoreAudio
0x000000001832f000  /System/Library/Frameworks/DiskArbitration.framework/Versions/A/DiskArbitration
0x000000001832f000  /usr/lib/liblangid.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/MultitouchSupport.framework/Versions/A/MultitouchSupport
0x000000001832f000  /System/Library/Frameworks/IOKit.framework/Versions/A/IOKit
0x000000001832f000  /usr/lib/libDiagnosticMessagesClient.dylib
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/CoreServices
0x000000001832f000  /System/Library/PrivateFrameworks/PerformanceAnalysis.framework/Versions/A/PerformanceAnalysis
0x000000001832f000  /System/Library/Frameworks/OpenGL.framework/Versions/A/OpenGL
0x000000001832f000  /System/Library/Frameworks/ColorSync.framework/Versions/A/ColorSync
0x000000001832f000  /System/Library/Frameworks/CoreImage.framework/Versions/A/CoreImage
0x000000001832f000  /System/Library/Frameworks/CoreText.framework/Versions/A/CoreText
0x000000001832f000  /System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO
0x000000001832f000  /System/Library/PrivateFrameworks/Backup.framework/Versions/A/Backup
0x000000001832f000  /usr/lib/libarchive.2.dylib
0x000000001832f000  /System/Library/Frameworks/CFNetwork.framework/Versions/A/CFNetwork
0x000000001832f000  /System/Library/Frameworks/SystemConfiguration.framework/Versions/A/SystemConfiguration
0x000000001832f000  /usr/lib/libCRFSuite.dylib
0x000000001832f000  /usr/lib/libc++.1.dylib
0x000000001832f000  /usr/lib/libc++abi.dylib
0x000000001832f000  /usr/lib/system/libcache.dylib
0x000000001832f000  /usr/lib/system/libcommonCrypto.dylib
0x000000001832f000  /usr/lib/system/libcompiler_rt.dylib
0x000000001832f000  /usr/lib/system/libcopyfile.dylib
0x000000001832f000  /usr/lib/system/libcorecrypto.dylib
0x000000001832f000  /usr/lib/system/libdispatch.dylib
0x000000001832f000  /usr/lib/system/libdyld.dylib
0x000000001832f000  /usr/lib/system/libkeymgr.dylib
0x000000001832f000  /usr/lib/system/liblaunch.dylib
0x000000001832f000  /usr/lib/system/libmacho.dylib
0x000000001832f000  /usr/lib/system/libquarantine.dylib
0x000000001832f000  /usr/lib/system/libremovefile.dylib
0x000000001832f000  /usr/lib/system/libsystem_asl.dylib
0x000000001832f000  /usr/lib/system/libsystem_blocks.dylib
0x000000001832f000  /usr/lib/system/libsystem_c.dylib
0x000000001832f000  /usr/lib/system/libsystem_configuration.dylib
0x000000001832f000  /usr/lib/system/libsystem_coreservices.dylib
0x000000001832f000  /usr/lib/system/libsystem_darwin.dylib
0x000000001832f000  /usr/lib/system/libsystem_dnssd.dylib
0x000000001832f000  /usr/lib/system/libsystem_info.dylib
0x000000001832f000  /usr/lib/system/libsystem_m.dylib
0x000000001832f000  /usr/lib/system/libsystem_malloc.dylib
0x000000001832f000  /usr/lib/system/libsystem_network.dylib
0x000000001832f000  /usr/lib/system/libsystem_networkextension.dylib
0x000000001832f000  /usr/lib/system/libsystem_notify.dylib
0x000000001832f000  /usr/lib/system/libsystem_sandbox.dylib
0x000000001832f000  /usr/lib/system/libsystem_secinit.dylib
0x000000001832f000  /usr/lib/system/libsystem_kernel.dylib
0x000000001832f000  /usr/lib/system/libsystem_platform.dylib
0x000000001832f000  /usr/lib/system/libsystem_pthread.dylib
0x000000001832f000  /usr/lib/system/libsystem_symptoms.dylib
0x000000001832f000  /usr/lib/system/libsystem_trace.dylib
0x000000001832f000  /usr/lib/system/libunwind.dylib
0x000000001832f000  /usr/lib/system/libxpc.dylib
0x000000001832f000  /usr/lib/closure/libclosured.dylib
0x000000001832f000  /usr/lib/libbsm.0.dylib
0x000000001832f000  /usr/lib/system/libkxld.dylib
0x000000001832f000  /usr/lib/libOpenScriptingUtil.dylib
0x000000001832f000  /usr/lib/libcoretls.dylib
0x000000001832f000  /usr/lib/libcoretls_cfhelpers.dylib
0x000000001832f000  /usr/lib/libpam.2.dylib
0x000000001832f000  /usr/lib/libsqlite3.dylib
0x000000001832f000  /usr/lib/libxar.1.dylib
0x000000001832f000  /usr/lib/libbz2.1.0.dylib
0x000000001832f000  /usr/lib/liblzma.5.dylib
0x000000001832f000  /usr/lib/libnetwork.dylib
0x000000001832f000  /usr/lib/libapple_nghttp2.dylib
0x000000001832f000  /usr/lib/libpcap.A.dylib
0x000000001832f000  /usr/lib/libboringssl.dylib
0x000000001832f000  /usr/lib/libusrtcp.dylib
0x000000001832f000  /usr/lib/libapple_crypto.dylib
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/FSEvents.framework/Versions/A/FSEvents
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/CarbonCore.framework/Versions/A/CarbonCore
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/Metadata.framework/Versions/A/Metadata
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/OSServices.framework/Versions/A/OSServices
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/SearchKit.framework/Versions/A/SearchKit
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/AE.framework/Versions/A/AE
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Versions/A/LaunchServices
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/DictionaryServices.framework/Versions/A/DictionaryServices
0x000000001832f000  /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/SharedFileList.framework/Versions/A/SharedFileList
0x000000001832f000  /System/Library/Frameworks/NetFS.framework/Versions/A/NetFS
0x000000001832f000  /System/Library/PrivateFrameworks/NetAuth.framework/Versions/A/NetAuth
0x000000001832f000  /System/Library/PrivateFrameworks/login.framework/Versions/A/Frameworks/loginsupport.framework/Versions/A/loginsupport
0x000000001832f000  /System/Library/PrivateFrameworks/TCC.framework/Versions/A/TCC
0x000000001832f000  /usr/lib/libmecabra.dylib
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/ATS.framework/Versions/A/ATS
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/ColorSyncLegacy.framework/Versions/A/ColorSyncLegacy
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/HIServices.framework/Versions/A/HIServices
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/LangAnalysis.framework/Versions/A/LangAnalysis
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/PrintCore.framework/Versions/A/PrintCore
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/QD.framework/Versions/A/QD
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/SpeechSynthesis.framework/Versions/A/SpeechSynthesis
0x000000001832f000  /System/Library/Frameworks/CoreDisplay.framework/Versions/A/CoreDisplay
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vImage.framework/Versions/A/vImage
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/vecLib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libvDSP.dylib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libBNNS.dylib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libQuadrature.dylib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libvMisc.dylib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libLAPACK.dylib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libBLAS.dylib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libLinearAlgebra.dylib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libSparse.dylib
0x000000001832f000  /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libSparseBLAS.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/IOAccelerator.framework/Versions/A/IOAccelerator
0x000000001832f000  /System/Library/PrivateFrameworks/IOPresentment.framework/Versions/A/IOPresentment
0x000000001832f000  /System/Library/PrivateFrameworks/DSExternalDisplay.framework/Versions/A/DSExternalDisplay
0x000000001832f000  /System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libCoreFSCache.dylib
0x000000001832f000  /System/Library/Frameworks/CoreVideo.framework/Versions/A/CoreVideo
0x000000001832f000  /System/Library/PrivateFrameworks/GraphVisualizer.framework/Versions/A/GraphVisualizer
0x000000001832f000  /System/Library/Frameworks/MetalPerformanceShaders.framework/Versions/A/MetalPerformanceShaders
0x000000001832f000  /usr/lib/libFosl_dynamic.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/FaceCore.framework/Versions/A/FaceCore
0x000000001832f000  /System/Library/Frameworks/OpenCL.framework/Versions/A/OpenCL
0x000000001832f000  /usr/lib/libcompression.dylib
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/ATS.framework/Versions/A/Resources/libFontParser.dylib
0x000000001832f000  /System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/ATS.framework/Versions/A/Resources/libFontRegistry.dylib
0x000000001832f000  /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libJPEG.dylib
0x000000001832f000  /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libTIFF.dylib
0x000000001832f000  /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libPng.dylib
0x000000001832f000  /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libGIF.dylib
0x000000001832f000  /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libJP2.dylib
0x000000001832f000  /System/Library/Frameworks/ImageIO.framework/Versions/A/Resources/libRadiance.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/AppleJPEG.framework/Versions/A/AppleJPEG
0x000000001832f000  /System/Library/PrivateFrameworks/MetalTools.framework/Versions/A/MetalTools
0x000000001832f000  /System/Library/Frameworks/MetalPerformanceShaders.framework/Frameworks/MPSCore.framework/Versions/A/MPSCore
0x000000001832f000  /System/Library/Frameworks/MetalPerformanceShaders.framework/Frameworks/MPSImage.framework/Versions/A/MPSImage
0x000000001832f000  /System/Library/Frameworks/MetalPerformanceShaders.framework/Frameworks/MPSMatrix.framework/Versions/A/MPSMatrix
0x000000001832f000  /System/Library/Frameworks/MetalPerformanceShaders.framework/Frameworks/MPSNeuralNetwork.framework/Versions/A/MPSNeuralNetwork
0x000000001832f000  /System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGLU.dylib
0x000000001832f000  /System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGFXShared.dylib
0x000000001832f000  /System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGL.dylib
0x000000001832f000  /System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGLImage.dylib
0x000000001832f000  /System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libCVMSPluginSupport.dylib
0x000000001832f000  /System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libCoreVMClient.dylib
0x000000001832f000  /usr/lib/libcups.2.dylib
0x000000001832f000  /System/Library/Frameworks/Kerberos.framework/Versions/A/Kerberos
0x000000001832f000  /System/Library/Frameworks/GSS.framework/Versions/A/GSS
0x000000001832f000  /usr/lib/libresolv.9.dylib
0x000000001832f000  /usr/lib/libiconv.2.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/Heimdal.framework/Versions/A/Heimdal
0x000000001832f000  /usr/lib/libheimdal-asn1.dylib
0x000000001832f000  /System/Library/Frameworks/OpenDirectory.framework/Versions/A/OpenDirectory
0x000000001832f000  /System/Library/PrivateFrameworks/CommonAuth.framework/Versions/A/CommonAuth
0x000000001832f000  /System/Library/Frameworks/OpenDirectory.framework/Versions/A/Frameworks/CFOpenDirectory.framework/Versions/A/CFOpenDirectory
0x000000001832f000  /System/Library/Frameworks/SecurityFoundation.framework/Versions/A/SecurityFoundation
0x000000001832f000  /System/Library/PrivateFrameworks/APFS.framework/Versions/A/APFS
0x000000001832f000  /usr/lib/libutil.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/AppleSauce.framework/Versions/A/AppleSauce
0x000000001832f000  /System/Library/PrivateFrameworks/LinguisticData.framework/Versions/A/LinguisticData
0x000000001832f000  /usr/lib/libmarisa.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/Lexicon.framework/Versions/A/Lexicon
0x000000001832f000  /usr/lib/libChineseTokenizer.dylib
0x000000001832f000  /usr/lib/libcmph.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/LanguageModeling.framework/Versions/A/LanguageModeling
0x000000001832f000  /System/Library/PrivateFrameworks/CoreEmoji.framework/Versions/A/CoreEmoji
0x000000001832f000  /System/Library/Frameworks/ServiceManagement.framework/Versions/A/ServiceManagement
0x000000001832f000  /System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Versions/A/BackgroundTaskManagement
0x000000001832f000  /usr/lib/libxslt.1.dylib
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/Ink.framework/Versions/A/Ink
0x000000001832f000  /System/Library/PrivateFrameworks/TextureIO.framework/Versions/A/TextureIO
0x000000001832f000  /usr/lib/libate.dylib
0x000000001832f000  /System/Library/PrivateFrameworks/CrashReporterSupport.framework/Versions/A/CrashReporterSupport
0x000000001832f000  /System/Library/PrivateFrameworks/Sharing.framework/Versions/A/Sharing
0x000000001832f000  /System/Library/PrivateFrameworks/IconServices.framework/Versions/A/IconServices
0x000000001832f000  /System/Library/PrivateFrameworks/ProtocolBuffer.framework/Versions/A/ProtocolBuffer
0x000000001832f000  /System/Library/PrivateFrameworks/Apple80211.framework/Versions/A/Apple80211
0x000000001832f000  /System/Library/Frameworks/CoreWLAN.framework/Versions/A/CoreWLAN
0x000000001832f000  /System/Library/PrivateFrameworks/CoreUtils.framework/Versions/A/CoreUtils
0x000000001832f000  /System/Library/Frameworks/IOBluetooth.framework/Versions/A/IOBluetooth
0x000000001832f000  /System/Library/PrivateFrameworks/CoreWiFi.framework/Versions/A/CoreWiFi
0x000000001832f000  /System/Library/Frameworks/CoreBluetooth.framework/Versions/A/CoreBluetooth
0x000000001832f000  /System/Library/PrivateFrameworks/SignpostNotification.framework/Versions/A/SignpostNotification
0x000000001832f000  /System/Library/PrivateFrameworks/DebugSymbols.framework/Versions/A/DebugSymbols
0x000000001832f000  /System/Library/PrivateFrameworks/CoreSymbolication.framework/Versions/A/CoreSymbolication
0x000000001832f000  /System/Library/PrivateFrameworks/Symbolication.framework/Versions/A/Symbolication
0x000000001832f000  /System/Library/PrivateFrameworks/AppleFSCompression.framework/Versions/A/AppleFSCompression
0x000000001832f000  /System/Library/PrivateFrameworks/SpeechRecognitionCore.framework/Versions/A/SpeechRecognitionCore
0x000000001832f000  /System/Library/CoreServices/Encodings/libSimplifiedChineseConverter.dylib
0x0000000100f3a000  /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/server/libjvm.dylib
0x000000001832f000  /usr/lib/libstdc++.6.0.9.dylib
0x0000000101ee1000  /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/libverify.dylib
0x0000000101eef000  /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/libjava.dylib
0x0000000101f25000  /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/libinstrument.dylib
0x0000000101fc3000  /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/libzip.dylib
0x000000001832f000  /System/Library/Frameworks/JavaVM.framework/Versions/A/Frameworks/JavaRuntimeSupport.framework/Versions/A/JavaRuntimeSupport
0x000000001832f000  /System/Library/Frameworks/JavaVM.framework/Versions/A/Frameworks/JavaNativeFoundation.framework/Versions/A/JavaNativeFoundation
0x000000001832f000  /System/Library/Frameworks/JavaVM.framework/Versions/A/JavaVM
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Carbon
0x000000001832f000  /System/Library/PrivateFrameworks/JavaLaunching.framework/Versions/A/JavaLaunching
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/CommonPanels.framework/Versions/A/CommonPanels
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/Help.framework/Versions/A/Help
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/ImageCapture.framework/Versions/A/ImageCapture
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/OpenScripting.framework/Versions/A/OpenScripting
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/Print.framework/Versions/A/Print
0x000000001832f000  /System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/SecurityHI.framework/Versions/A/SecurityHI
0x000000010ae78000  /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/libnet.dylib
0x000000010aed7000  /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/libmanagement.dylib
0x000000010aee5000  /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/libnio.dylib

VM Arguments:
jvm_args: -XX:TieredStopAtLevel=1 -Xverify:none -Dspring.output.ansi.enabled=always -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=52007 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dspring.liveBeansView.mbeanDomain -Dspring.application.admin.enabled=true -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=52008:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 
java_command: com.ejlerp.saleorder.SaleOrderProviderApplication
java_class_path (initial): /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/charsets.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/deploy.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/cldrdata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/dnsns.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/jaccess.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/jfxrt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/localedata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/nashorn.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/sunec.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/ext/zipfs.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/javaws.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/jce.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/jfr.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/jfxswt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/jsse.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/management-agent.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/plugin.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/rt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/lib/ant-javafx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home
Launcher Type: SUN_STANDARD

Environment Variables:
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home
PATH=/Users/victor/jar/byteman-download-3.0.10/bin:/Users/victor/jar/btrace-bin-1.3.10/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
SHELL=/bin/bash

Signal Handlers:
SIGSEGV: [libjvm.dylib+0x5b27a5], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_ONSTACK|SA_RESTART|SA_SIGINFO
SIGBUS: [libjvm.dylib+0x5b27a5], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_RESTART|SA_SIGINFO
SIGFPE: [libjvm.dylib+0x4891c4], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_RESTART|SA_SIGINFO
SIGPIPE: [libjvm.dylib+0x4891c4], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_RESTART|SA_SIGINFO
SIGXFSZ: [libjvm.dylib+0x4891c4], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_RESTART|SA_SIGINFO
SIGILL: [libjvm.dylib+0x4891c4], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_RESTART|SA_SIGINFO
SIGUSR1: SIG_DFL, sa_mask[0]=00000000000000000000000000000000, sa_flags=none
SIGUSR2: [libjvm.dylib+0x488ce2], sa_mask[0]=00100000000000000000000000000000, sa_flags=SA_RESTART|SA_SIGINFO
SIGHUP: [libjvm.dylib+0x4872b9], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_RESTART|SA_SIGINFO
SIGINT: [libjvm.dylib+0x4872b9], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_RESTART|SA_SIGINFO
SIGTERM: [libjvm.dylib+0x4872b9], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_RESTART|SA_SIGINFO
SIGQUIT: [libjvm.dylib+0x4872b9], sa_mask[0]=11111111011111110111111111111111, sa_flags=SA_RESTART|SA_SIGINFO


---------------  S Y S T E M  ---------------

OS:Bsduname:Darwin 17.3.0 Darwin Kernel Version 17.3.0: Thu Nov  9 18:09:22 PST 2017; root:xnu-4570.31.3~1/RELEASE_X86_64 x86_64
rlimit: STACK 8192k, CORE 0k, NPROC 709, NOFILE 10240, AS infinity
load average:5.50 4.41 3.51

CPU:total 4 (initial active 4) (2 cores per cpu, 2 threads per core) family 6 model 61 stepping 4, cmov, cx8, fxsr, mmx, sse, sse2, sse3, ssse3, sse4.1, sse4.2, popcnt, avx, avx2, aes, clmul, erms, 3dnowpref, lzcnt, ht, tsc, tscinvbit, bmi1, bmi2, adx

Memory: 4k page, physical 8388608k(184936k free)

/proc/meminfo:


vm_info: Java HotSpot(TM) 64-Bit Server VM (25.151-b12) for bsd-amd64 JRE (1.8.0_151-b12), built on Sep  5 2017 19:37:08 by "java_re" with gcc 4.2.1 (Based on Apple Inc. build 5658) (LLVM build 2336.11.00)

time: Thu Dec 28 15:17:01 2017
elapsed time: 72 seconds (0d 0h 1m 12s)



```