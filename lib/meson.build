sources = ['stdatomic.c', 'atomic.S', 'builtin.c']

if host_machine.system() == 'linux'
    sources += ['dsohandle.c']
endif

if host_machine.system() == 'windows'
    #I cannot find where these are located.
    #sources += ['crt1.o crt1w.o wincrt1.o wincrt1w.o dllcrt1.o dllmain.o']
endif

if host_machine.cpu_family() == 'x86' or host_machine.cpu_family() == 'amd64'
    sources += [
        'libtcc1.c',
        'alloca.S',
        'alloca-bt.S',
    ]
endif

if host_machine.cpu_family() == 'arm'
    sources += [
        'libtcc1.c',
        'armeabi.c',
        'alloca.S',
        'armflush.c',
    ]
endif

if host_machine.cpu_family() == 'aarch64' or host_machine.cpu_family() == 'risc64'
    sources += [
        'lib-arm64.c',
    ]
endif

libtcc1 = static_library('tcc1', pic: true, install: true, sources: sources)