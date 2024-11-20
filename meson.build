project(
  'tcc',
  'c',
  version: '0.9.28vs-rc',
  meson_version: '>= 1.1',
)

cc = meson.get_compiler('c')

sources = [
  'libtcc.c',
  'tccpp.c',
  'tccgen.c',
  'tccelf.c',
  'tccasm.c',
  'tccdbg.c',
  'tccrun.c',
]

cdata = configuration_data()
cdata.set_quoted('TCC_VERSION', meson.project_version())

tccdir = get_option('prefix') / get_option('libdir') / 'tcc'
cdata.set_quoted('CONFIG_TCCDIR', tccdir)

host_os = host_machine.system()
if host_os == 'windows'
  sources += 'tccpe.c'
  cdata.set('TCC_TARGET_PE', true)
elif host_os == 'darwin'
  sources += 'tccmacho.c'
  cdata.set('TCC_TARGET_MACHO', true)
endif

host_arch = host_machine.cpu_family()
if host_arch == 'x86'
  sources += [
    'i386-gen.c',
    'i386-link.c',
    'i386-asm.c',
  ]

  cdata.set('TCC_TARGET_I386', true)
elif host_arch == 'x86_64'
  sources += [
    'x86_64-gen.c',
    'x86_64-link.c',
    'i386-asm.c',
  ]
  if host_os != 'windows'
    sources += 'lib/va_list.c'
  endif

  cdata.set('TCC_TARGET_X86_64', true)
elif host_arch == 'risc64'
  sources += [
    'riscv64-gen.c', 
    'riscv64-link.c', 
    'riscv64-asm.c'
  ]
  cdata.set('TCC_TARGET_RISCV64', true)

elif host_arch == 'arm'
  sources += [
    'arm-gen.c',
    'arm-link.c',
    'arm-asm.c',
  ]

  cdata.set('TCC_TARGET_ARM', true)
  cdata.set('TCC_ARM_EABI', true)
  cdata.set('TCC_ARM_VFP', true)

  is_hardfloat_src = '''
    #ifndef __ARM_PCS_VFP
    # error Not hardfloat
    #endif
  '''
  if cc.compiles(is_hardfloat_src, name: 'hardfloat ABI')
    cdata.set('TCC_ARM_HARDFLOAT', true)
  endif
elif host_arch == 'aarch64'
  sources += [
    'arm64-gen.c',
    'arm64-link.c',
    'arm-asm.c',
  ]

  cdata.set('TCC_TARGET_ARM64', true)

  have_ptrauth_src = '''
    #ifdef __clang__
    # if __has_feature(ptrauth_calls)
    #  define HAVE_PTRAUTH 1
    # endif
    #endif

    #ifndef HAVE_PTRAUTH
    # error Pointer authentication not supported
    #endif
  '''
  if cc.compiles(have_ptrauth_src, name: 'pointer authentication')
    cdata.set('HAVE_PTRAUTH', true)
  endif
endif

install_headers('libtcc.h')

runtime_headers = [
  'include/float.h',
  'include/stdalign.h',
  'include/stdarg.h',
  'include/stdatomic.h',
  'include/stdbool.h',
  'include/stddef.h',
  'include/stdnoreturn.h',
  'include/tccdefs.h',
  'include/tgmath.h',
  'include/varargs.h',
]
install_headers(runtime_headers, install_dir: tccdir / 'include')

configure_file(
  output: 'config.h',
  configuration: cdata,
)

threads_dep = dependency('threads')
dl_dep = cc.find_library('dl', required: false)


libtcc = library(
  'tcc',
  sources,
  c_args: ['-DONE_SOURCE=0'],
  install: true,
  dependencies: [threads_dep, dl_dep],
)

libtcc_dep = declare_dependency(
  link_with: libtcc,
  include_directories: include_directories('.'),
  variables: {'libdir': get_option('libdir')},
)

tcc = executable(
  'tcc',
  sources: ['tcc.c'],
  link_with: libtcc,
  install: not meson.is_subproject(),
)

makeinfo = find_program('makeinfo',required:false)
if makeinfo.found()
#  custom_target('docs-html', output: 'tcc-doc.html', input: 'tcc-doc.texi', command:['makeinfo','@INPUT@','--no-split','--html','--number-sections','-o','@OUTPUT@'], install: not meson.is_subproject(), install_dir : get_option('datadir'))
#  custom_target('docs-info', output: 'tcc-doc.info', input: 'tcc-doc.texi', command:['makeinfo','@INPUT@','-o','@OUTPUT@'], install: not meson.is_subproject(), install_dir : get_option('infodir'))
endif

#perl = find_program('perl', required:false)
#if perl.found()
#  pod = custom_target('docs-pod', output: 'tcc-doc.pod', input: 'tcc-doc.texi', command:['perl','./texi2pod.pl','@INPUT@','@OUTPUT@'])
#  #TODO Add entry for common.texi
#
#  pod2man = find_program('pod2man', required:false)
#  if pod2man.found()
#    #custom_target('docs-man', depends: [pod], output: 'tcc.doc.1', input: 'tcc-doc.pod', command:['pod2man','--section=1', '--center="Tiny C Compiler"', '--release="'+meson.project_version()+'"','@INPUT@','@OUTPUT@'], install: not meson.is_subproject(), install_dir : get_option('mandir'))
#  endif 
#endif 


if get_option('tests')
    subdir(['./tests/'])
endif

pkg = import('pkgconfig')
pkg.generate(
  libtcc,
  name: 'libtcc',
  description: 'Tiny C Compiler',
  url: 'https://bellard.org/tcc/',
)

meson.override_dependency('libtcc', libtcc_dep)
