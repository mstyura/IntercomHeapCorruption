# Demo of heap corruption when intercom library is linked into application.

## Steps to reproduce
1. Compile demo application from this repository;
2. Run it on device;

## Actual result

Around 10% of launches heap corruption checker is triggered;

## Expected result

Heap corruption checker does not detect corrupted memory (because memory should not be corrupted, not the checker is wrong).

## Additional info

Demo application is empty project with single SPM dependency of https://github.com/intercom/intercom-ios/tree/16.1.1 (issue is also presented at least since version of `10.3.2`).
It is configured to compile in release mode. 
`libmalloc` checks are enabled after each manipulation with heap (after each call to `malloc` & `free`).

`libmalloc` heap checker is unable to detect problem when app is launches on simulator;
`libgmalloc` (guarded malloc) is also unable to detect problem when launched on simulator (`libgmalloc` is only available on simulator);
`asan` is unable to detect memory problem neither on device or simulator.


Here is and **example** of debugger output when issue is reproduced during launch (it might be different from launch to launch due to nature of a problem: memory could be corrupted in one place, but detection could happen in other place):
```log
IntercomHeapCorruption(3588,0x1f2125900) malloc: Unrecognized value for MallocDebugReport (1) - using 'stderr'
IntercomHeapCorruption(3588,0x1f2125900) malloc: enabling abort() on bad malloc or free
IntercomHeapCorruption(3588,0x1f2125900) malloc: checks heap after operation #1 and each 1 operations
IntercomHeapCorruption(3588,0x1f2125900) malloc: will abort on heap corruption
IntercomHeapCorruption(3588,0x1f2125900) malloc: environment variables that can be set for debug:
- MallocLogFile <f> to create/append messages to file <f> instead of stderr
- MallocGuardEdges to add 2 guard pages for each large block
- MallocDoNotProtectPrelude to disable protection (when previous flag set)
- MallocDoNotProtectPostlude to disable protection (when previous flag set)
- MallocStackLogging to record all stacks.  Tools like leaks can then be applied
- MallocStackLoggingNoCompact to record all stacks.  Needed for malloc_history
- MallocStackLoggingDirectory to set location of stack logs, which can grow large; default is /tmp
- MallocScribble to detect writing on free blocks and missing initializers:
  0x55 is written upon free and 0xaa is written on allocation
- MallocCheckHeapStart <n> to start checking the heap after <n> operations
- MallocCheckHeapEach <s> to repeat the checking of the heap after <s> operations
- MallocCheckHeapSleep <t> to sleep <t> seconds on heap corruption
- MallocCheckHeapAbort <b> to abort on heap corruption if <b> is non-zero
- MallocCorruptionAbort to abort on malloc errors, but not on out of memory for 32-bit processes
  MallocCorruptionAbort is always set on 64-bit processes
- MallocErrorAbort to abort on any malloc error, including out of memory
- MallocTracing to emit kdebug trace points on malloc entry points
- MallocHelp - this help!
IntercomHeapCorruption(3588,0x16b2cb000) malloc: at szone_check counter=10000
IntercomHeapCorruption(3588,0x16b2cb000) malloc: *** MallocCheckHeap: FAILED check at operation #11112
Stack for last operation where the malloc check succeeded: 0x1a9ecd7c4 0x1a9ec8c88 0x1e8b6bf88 0x1e8b81588 0x1e8b6e94c 0x1e8b6bf88 0x1e8b81588 0x1e8b6c004 0x1e8b81588 0x194ddd4a4 0x194de2298 0x194deb994 0x1a9082cdc 0x1a907698c 0x1a907ff38 0x1a907baa8 0x1a30884b4 0x1a3089fdc 0x1a3091694 0x1a3092214 0x1a309ce10 0x1e8b13df8 0x1e8b13b98 
(Use 'atos' for a symbolic stack)
IntercomHeapCorruption(3588,0x16b2cb000) malloc: *** check: incorrect tiny region 18, counter=11106
*** invariant broken for tiny block 0x104f0b140 this msize=0 - size is too small
IntercomHeapCorruption(3588,0x16b2cb000) malloc: *** set a breakpoint in malloc_error_break to debug
(lldb) bt all
  thread #1, queue = 'com.apple.uikit.applicationSupportClient'
    frame #0: 0x00000001d86b2680 libsystem_kernel.dylib`__ulock_wait + 8
    frame #1: 0x00000001e8a7e82c libsystem_platform.dylib`_os_unfair_lock_lock_slow + 172
    frame #2: 0x00000001b177ee4c BoardServices`-[BSXPCServiceConnection activateNowOrWhenReady:] + 92
    frame #3: 0x00000001b177f174 BoardServices`-[BSServiceConnection activate] + 696
    frame #4: 0x00000001a8cee94c UIKitServices`__44-[UISApplicationSupportClient _remoteTarget]_block_invoke + 288
    frame #5: 0x00000001a3089fdc libdispatch.dylib`_dispatch_client_callout + 20
    frame #6: 0x00000001a3099574 libdispatch.dylib`_dispatch_lane_barrier_sync_invoke_and_complete + 56
    frame #7: 0x00000001a8cef264 UIKitServices`-[UISApplicationSupportClient _remoteTarget] + 184
    frame #8: 0x00000001a8cef0b8 UIKitServices`-[UISApplicationSupportClient applicationInitializationContextWithParameters:] + 164
    frame #9: 0x000000019e00ee74 UIKitCore`__63-[_UIApplicationConfigurationLoader _loadInitializationContext]_block_invoke_2 + 172
    frame #10: 0x000000019e0fb52c UIKitCore`__UIAPPLICATION_IS_LOADING_INITIALIZATION_INFO_FROM_THE_SYSTEM__ + 28
    frame #11: 0x000000019e17503c UIKitCore`__63-[_UIApplicationConfigurationLoader _loadInitializationContext]_block_invoke + 100
    frame #12: 0x00000001a3089fdc libdispatch.dylib`_dispatch_client_callout + 20
    frame #13: 0x00000001a308b828 libdispatch.dylib`_dispatch_once_callout + 32
    frame #14: 0x000000019dde547c UIKitCore`-[_UIApplicationConfigurationLoader _loadInitializationContext] + 152
    frame #15: 0x000000019dde4bb0 UIKitCore`-[_UIApplicationConfigurationLoader applicationInitializationContext] + 24
    frame #16: 0x000000019e010bb4 UIKitCore`-[_UIDeviceInitialDeviceConfigurationLoader initialDeviceContext] + 92
    frame #17: 0x000000019e0ec5cc UIKitCore`___UIDeviceNativeUserInterfaceIdiom_block_invoke + 60
    frame #18: 0x00000001a3089fdc libdispatch.dylib`_dispatch_client_callout + 20
    frame #19: 0x00000001a308b828 libdispatch.dylib`_dispatch_once_callout + 32
    frame #20: 0x000000019e1350f8 UIKitCore`__initializeActiveUserInterfaceIdiom_block_invoke + 72
    frame #21: 0x00000001a3089fdc libdispatch.dylib`_dispatch_client_callout + 20
    frame #22: 0x00000001a308b828 libdispatch.dylib`_dispatch_once_callout + 32
    frame #23: 0x000000019dd79ca4 UIKitCore`-[UIDevice userInterfaceIdiom] + 64
    frame #24: 0x000000019def2a44 UIKitCore`-[UIBarAppearance init] + 44
    frame #25: 0x00000001053b4b88 Intercom`+[ICMAppearanceReset resetAppearanceWithin:] + 640
    frame #26: 0x000000010535b6c4 Intercom`-[ICMPresentationManager init] + 212
    frame #27: 0x000000010535b544 Intercom`__40+[ICMPresentationManager sharedInstance]_block_invoke + 20
    frame #28: 0x00000001a3089fdc libdispatch.dylib`_dispatch_client_callout + 20
    frame #29: 0x00000001a308b828 libdispatch.dylib`_dispatch_once_callout + 32
    frame #30: 0x000000010535b52c Intercom`+[ICMPresentationManager sharedInstance] + 60
    frame #31: 0x000000010535b59c Intercom`+[ICMPresentationManager observeSceneWillEnterForeground] + 64
    frame #32: 0x0000000194deca80 libobjc.A.dylib`load_images + 824
    frame #33: 0x00000001ba18c3fc dyld`dyld4::RuntimeState::notifyObjCInit(dyld4::Loader const*) + 164
    frame #34: 0x00000001ba18fcc0 dyld`dyld4::Loader::runInitializersBottomUp(dyld4::RuntimeState&, dyld3::Array<dyld4::Loader const*>&) const + 204
    frame #35: 0x00000001ba18fca8 dyld`dyld4::Loader::runInitializersBottomUp(dyld4::RuntimeState&, dyld3::Array<dyld4::Loader const*>&) const + 180
    frame #36: 0x00000001ba19533c dyld`dyld4::Loader::runInitializersBottomUpPlusUpwardLinks(dyld4::RuntimeState&) const + 328
    frame #37: 0x00000001ba1c9244 dyld`dyld4::APIs::runAllInitializersForMain() + 360
    frame #38: 0x00000001ba19e66c dyld`dyld4::prepare(dyld4::APIs&, dyld3::MachOAnalyzer const*) + 3388
    frame #39: 0x00000001ba19c8d4 dyld`start + 2388
  thread #2, queue = 'com.apple.root.user-initiated-qos'
    frame #0: 0x00000001d86b2680 libsystem_kernel.dylib`__ulock_wait + 8
    frame #1: 0x00000001a308a9cc libdispatch.dylib`_dlock_wait + 56
    frame #2: 0x00000001a308a8fc libdispatch.dylib`_dispatch_once_wait + 112
    frame #3: 0x000000019dde547c UIKitCore`-[_UIApplicationConfigurationLoader _loadInitializationContext] + 152
    frame #4: 0x000000019e04377c UIKitCore`__70-[_UIApplicationConfigurationLoader startPreloadInitializationContext]_block_invoke + 20
    frame #5: 0x00000001a30884b4 libdispatch.dylib`_dispatch_call_block_and_release + 32
    frame #6: 0x00000001a3089fdc libdispatch.dylib`_dispatch_client_callout + 20
    frame #7: 0x00000001a309bb8c libdispatch.dylib`_dispatch_root_queue_drain + 684
    frame #8: 0x00000001a309c284 libdispatch.dylib`_dispatch_worker_thread2 + 164
    frame #9: 0x00000001e8b13dbc libsystem_pthread.dylib`_pthread_wqthread + 228
* thread #3, queue = 'com.apple.runningboardservices.connection.outgoing', stop reason = breakpoint 1.1
  * frame #0: 0x00000001a9ec8478 libsystem_malloc.dylib`malloc_error_break
    frame #1: 0x00000001a9ecce98 libsystem_malloc.dylib`malloc_vreport + 760
    frame #2: 0x00000001a9ec89e8 libsystem_malloc.dylib`malloc_zone_check_fail + 604
    frame #3: 0x00000001a9ecb758 libsystem_malloc.dylib`tiny_check_region + 1572
    frame #4: 0x00000001a9ecc9c8 libsystem_malloc.dylib`tiny_check + 264
    frame #5: 0x00000001a9ec11f0 libsystem_malloc.dylib`szone_check_all + 60
    frame #6: 0x00000001a9ec9268 libsystem_malloc.dylib`malloc_zone_check + 168
    frame #7: 0x00000001a9ecd76c libsystem_malloc.dylib`internal_check + 60
    frame #8: 0x00000001a9ec8c88 libsystem_malloc.dylib`find_zone_and_free + 644
    frame #9: 0x00000001e8b81588 libxpc.dylib`-[OS_xpc_object dealloc] + 28
    frame #10: 0x00000001e8b6e94c libxpc.dylib`_xpc_dictionary_node_free + 84
    frame #11: 0x00000001e8b6bf88 libxpc.dylib`_xpc_dictionary_dispose + 48
    frame #12: 0x00000001e8b81588 libxpc.dylib`-[OS_xpc_object dealloc] + 28
    frame #13: 0x00000001e8b6e94c libxpc.dylib`_xpc_dictionary_node_free + 84
    frame #14: 0x00000001e8b6bf88 libxpc.dylib`_xpc_dictionary_dispose + 48
    frame #15: 0x00000001e8b81588 libxpc.dylib`-[OS_xpc_object dealloc] + 28
    frame #16: 0x00000001e8b6c004 libxpc.dylib`_xpc_dictionary_dispose + 172
    frame #17: 0x00000001e8b81588 libxpc.dylib`-[OS_xpc_object dealloc] + 28
    frame #18: 0x0000000194ddd4a4 libobjc.A.dylib`object_cxxDestructFromClass(objc_object*, objc_class*) + 116
    frame #19: 0x0000000194de2298 libobjc.A.dylib`objc_destructInstance + 80
    frame #20: 0x0000000194deb994 libobjc.A.dylib`_objc_rootDealloc + 80
    frame #21: 0x00000001a9082cdc RunningBoardServices`-[RBSXPCCoder dealloc] + 112
    frame #22: 0x00000001a907698c RunningBoardServices`-[RBSXPCMessage invokeOnConnection:withReturnCollectionClass:entryClass:error:] + 348
    frame #23: 0x00000001a907ff38 RunningBoardServices`-[RBSXPCMessage invokeOnConnection:withReturnClass:error:] + 32
    frame #24: 0x00000001a907baa8 RunningBoardServices`__27-[RBSConnection _handshake]_block_invoke + 348
    frame #25: 0x00000001a30884b4 libdispatch.dylib`_dispatch_call_block_and_release + 32
    frame #26: 0x00000001a3089fdc libdispatch.dylib`_dispatch_client_callout + 20
    frame #27: 0x00000001a3091694 libdispatch.dylib`_dispatch_lane_serial_drain + 672
    frame #28: 0x00000001a3092214 libdispatch.dylib`_dispatch_lane_invoke + 436
    frame #29: 0x00000001a309ce10 libdispatch.dylib`_dispatch_workloop_worker_thread + 652
    frame #30: 0x00000001e8b13df8 libsystem_pthread.dylib`_pthread_wqthread + 288
  thread #4, queue = 'BSXPCCnx:com.apple.frontboard.systemappservices'
    frame #0: 0x00000001d86b2680 libsystem_kernel.dylib`__ulock_wait + 8
    frame #1: 0x00000001e8a7e82c libsystem_platform.dylib`_os_unfair_lock_lock_slow + 172
    frame #2: 0x00000001a9eb0d84 libsystem_malloc.dylib`tiny_madvise_free_range_no_lock + 272
    frame #3: 0x00000001a9eb30d0 libsystem_malloc.dylib`tiny_free_no_lock + 520
    frame #4: 0x00000001a9eb3c18 libsystem_malloc.dylib`free_tiny + 496
    frame #5: 0x00000001e8b815a4 libxpc.dylib`-[OS_xpc_object dealloc] + 56
    frame #6: 0x00000001e8b666b8 libxpc.dylib`xpc_connection_copy_bundle_id + 80
    frame #7: 0x00000001a3202498 BaseBoard`_BSBundleIDForXPCConnectionAndIKnowWhatImDoingISwear + 40
    frame #8: 0x00000001a31dda34 BaseBoard`+[BSProcessHandle processHandleForXPCConnection:] + 100
    frame #9: 0x00000001b178eb0c BoardServices`+[BSXPCServiceConnectionPeer peerOfConnection:] + 320
    frame #10: 0x00000001b17b8194 BoardServices`__55-[BSXPCServiceConnection _lock_activateNowOrWhenReady:]_block_invoke_3 + 136
    frame #11: 0x00000001b1783ef8 BoardServices`__52-[BSXPCServiceConnectionMessage _sendSynchronously:]_block_invoke + 216
    frame #12: 0x00000001e8b74f1c libxpc.dylib`_xpc_connection_reply_callout + 124
    frame #13: 0x00000001e8b67fb4 libxpc.dylib`_xpc_connection_call_reply_async + 88
    frame #14: 0x00000001a308a05c libdispatch.dylib`_dispatch_client_callout3 + 20
    frame #15: 0x00000001a30a7f58 libdispatch.dylib`_dispatch_mach_msg_async_reply_invoke + 344
    frame #16: 0x00000001a309156c libdispatch.dylib`_dispatch_lane_serial_drain + 376
    frame #17: 0x00000001a3092214 libdispatch.dylib`_dispatch_lane_invoke + 436
    frame #18: 0x00000001a309ce10 libdispatch.dylib`_dispatch_workloop_worker_thread + 652
    frame #19: 0x00000001e8b13df8 libsystem_pthread.dylib`_pthread_wqthread + 288
  thread #5
    frame #0: 0x00000001d86b2050 libsystem_kernel.dylib`__workq_kernreturn + 8
  thread #6
    frame #0: 0x00000001d86b2050 libsystem_kernel.dylib`__workq_kernreturn + 8
(lldb) image lookup --address 0x1a9ecd7c4
image lookup --address 0x1a9ec8c88
image lookup --address 0x1e8b6bf88
image lookup --address 0x1e8b81588
image lookup --address 0x1e8b6e94c
image lookup --address 0x1e8b6bf88
image lookup --address 0x1e8b81588
image lookup --address 0x1e8b6c004
image lookup --address 0x1e8b81588
image lookup --address 0x194ddd4a4
image lookup --address 0x194de2298
image lookup --address 0x194deb994
image lookup --address 0x1a9082cdc
image lookup --address 0x1a907698c
image lookup --address 0x1a907ff38
image lookup --address 0x1a907baa8
image lookup --address 0x1a30884b4
image lookup --address 0x1a3089fdc
image lookup --address 0x1a3091694
image lookup --address 0x1a3092214
image lookup --address 0x1a309ce10
image lookup --address 0x1e8b13df8
image lookup --address 0x1e8b13b98
      Address: libsystem_malloc.dylib[0x00000001951a97c4] (libsystem_malloc.dylib.__TEXT.__text + 118868)
      Summary: libsystem_malloc.dylib`internal_check + 148
      Address: libsystem_malloc.dylib[0x00000001951a4c88] (libsystem_malloc.dylib.__TEXT.__text + 99608)
      Summary: libsystem_malloc.dylib`find_zone_and_free + 644
(lldb)       Address: libxpc.dylib[0x00000001d3e47f88] (libxpc.dylib.__TEXT.__text + 78012)
      Summary: libxpc.dylib`_xpc_dictionary_dispose + 48
(lldb)       Address: libxpc.dylib[0x00000001d3e5d588] (libxpc.dylib.__TEXT.__text + 165564)
      Summary: libxpc.dylib`-[OS_xpc_object dealloc] + 28
(lldb)       Address: libxpc.dylib[0x00000001d3e4a94c] (libxpc.dylib.__TEXT.__text + 88704)
      Summary: libxpc.dylib`_xpc_dictionary_node_free + 84
(lldb)       Address: libxpc.dylib[0x00000001d3e47f88] (libxpc.dylib.__TEXT.__text + 78012)
      Summary: libxpc.dylib`_xpc_dictionary_dispose + 48
(lldb)       Address: libxpc.dylib[0x00000001d3e5d588] (libxpc.dylib.__TEXT.__text + 165564)
      Summary: libxpc.dylib`-[OS_xpc_object dealloc] + 28
(lldb)       Address: libxpc.dylib[0x00000001d3e48004] (libxpc.dylib.__TEXT.__text + 78136)
      Summary: libxpc.dylib`_xpc_dictionary_dispose + 172
      Address: libxpc.dylib[0x00000001d3e5d588] (libxpc.dylib.__TEXT.__text + 165564)
      Summary: libxpc.dylib`-[OS_xpc_object dealloc] + 28
      Address: libobjc.A.dylib[0x00000001800b94a4] (libobjc.A.dylib.__TEXT.__text + 164)
      Summary: libobjc.A.dylib`object_cxxDestructFromClass(objc_object*, objc_class*) + 116
      Address: libobjc.A.dylib[0x00000001800be298] (libobjc.A.dylib.__TEXT.__text + 20120)
      Summary: libobjc.A.dylib`objc_destructInstance + 80
      Address: libobjc.A.dylib[0x00000001800c7994] (libobjc.A.dylib.__TEXT.__text + 58772)
      Summary: libobjc.A.dylib`_objc_rootDealloc + 80
(lldb)       Address: RunningBoardServices[0x000000019435ecdc] (RunningBoardServices.__TEXT.__text + 62784)
      Summary: RunningBoardServices`-[RBSXPCCoder dealloc] + 112
(lldb)       Address: RunningBoardServices[0x000000019435298c] (RunningBoardServices.__TEXT.__text + 12784)
      Summary: RunningBoardServices`-[RBSXPCMessage invokeOnConnection:withReturnCollectionClass:entryClass:error:] + 348
(lldb)       Address: RunningBoardServices[0x000000019435bf38] (RunningBoardServices.__TEXT.__text + 51100)
      Summary: RunningBoardServices`-[RBSXPCMessage invokeOnConnection:withReturnClass:error:] + 32
(lldb)       Address: RunningBoardServices[0x0000000194357aa8] (RunningBoardServices.__TEXT.__text + 33548)
      Summary: RunningBoardServices`__27-[RBSConnection _handshake]_block_invoke + 348
(lldb)       Address: libdispatch.dylib[0x000000018e3644b4] (libdispatch.dylib.__TEXT.__text + 2600)
      Summary: libdispatch.dylib`_dispatch_call_block_and_release + 32
      Address: libdispatch.dylib[0x000000018e365fdc] (libdispatch.dylib.__TEXT.__text + 9552)
      Summary: libdispatch.dylib`_dispatch_client_callout + 20
      Address: libdispatch.dylib[0x000000018e36d694] (libdispatch.dylib.__TEXT.__text + 39944)
      Summary: libdispatch.dylib`_dispatch_lane_serial_drain + 672
      Address: libdispatch.dylib[0x000000018e36e214] (libdispatch.dylib.__TEXT.__text + 42888)
      Summary: libdispatch.dylib`_dispatch_lane_invoke + 436
      Address: libdispatch.dylib[0x000000018e378e10] (libdispatch.dylib.__TEXT.__text + 86916)
      Summary: libdispatch.dylib`_dispatch_workloop_worker_thread + 652
(lldb)       Address: libsystem_pthread.dylib[0x00000001d3defdf8] (libsystem_pthread.dylib.__TEXT.__text + 872)
      Summary: libsystem_pthread.dylib`_pthread_wqthread + 288
      Address: libsystem_pthread.dylib[0x00000001d3defb98] (libsystem_pthread.dylib.__TEXT.__text + 264)
      Summary: libsystem_pthread.dylib`start_wqthread + 8
(lldb) 
```