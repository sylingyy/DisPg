Turn off PatchGuard in real time for win7 (7600) ~ win10 (17134).
 
![Image text](https://github.com/9176324/DisPg/blob/master/attach%20PatchGuard.png)
 
Test for BigPool.
 
![Image text](https://github.com/9176324/DisPg/blob/master/BigPool.png)
 
Test for pages and worker.
 
![Image text](https://github.com/9176324/DisPg/blob/master/PagesAndWorker.png)
  
  
```
typedef struct _CALLERS {
    PVOID * EstablisherFrame;
    PVOID Establisher;
}CALLERS, *PCALLERS;

DECLSPEC_NOINLINE
ULONG
NTAPI
WalkFrameChain(
    __out PCALLERS Callers,
    __in ULONG Count
)
{
    CONTEXT ContextRecord = { 0 };
    PVOID HandlerData = NULL;
    ULONG Index = 0;
    PRUNTIME_FUNCTION FunctionEntry = NULL;
    ULONG64 ImageBase = 0;
    ULONG64 EstablisherFrame = 0;

    RtlCaptureContext(&ContextRecord);

    __try {
        while (Index < Count &&
            0 != ContextRecord.Rip) {
            FunctionEntry = DetourRtlLookupFunctionEntry(
                ContextRecord.Rip,
                &ImageBase,
                NULL);

            if (NULL != FunctionEntry) {
                RtlVirtualUnwind(
                    UNW_FLAG_NHANDLER,
                    ImageBase,
                    ContextRecord.Rip,
                    FunctionEntry,
                    &ContextRecord,
                    &HandlerData,
                    &EstablisherFrame,
                    NULL);

                Callers[Index].Establisher = (PVOID)ContextRecord.Rip;
                Callers[Index].EstablisherFrame = (PVOID *)EstablisherFrame;

                Index += 1;
            }
            else {
                break;
            }
        }
    }
    __except (EXCEPTION_EXECUTE_HANDLER) {
        Index = 0;
    }

    return Index;
}
```

