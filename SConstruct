import os

DIRECTORIES = [
    'src',
    'components/rotatable' ]

def target_architectures():
    archs = os.getenv('FSARCHS', None)
    if archs:
        return archs.split(',')

    arch_map = {
        ('Darwin', 'arm64'): ['darwin'],
        ('Darwin', 'x86_64'): ['darwin'],
        ('FreeBSD', 'amd64'): ['freebsd_amd64'],
        ('Linux', 'i686'): ['linux32'],
        ('Linux', 'x86_64'): ['linux64'],
        ('OpenBSD', 'amd64'): ['openbsd_amd64'],
    }

    uname_os, _, _, _, uname_cpu = os.uname()
    assert (uname_os, uname_cpu) in arch_map
    return arch_map[(uname_os, uname_cpu)]

TARGET_DEFINES = {
    'freebsd_amd64': [],
    'linux32': ['_FILE_OFFSET_BITS=64'],
    'linux64': [],
    'openbsd_amd64': [],
    'darwin': []
}

TARGET_FLAGS = {
    'freebsd_amd64': '',
    'linux32': '-m32 ',
    'linux64': '',
    'openbsd_amd64': '',
    'darwin': '-mmacosx-version-min=10.13 '
}

def libconfig_builder(env):
    env.Install('.', '#fscomp-libconfig.json')

def libconfig_parser():
    return '$ARCHBUILDDIR/components/rotatable/.fscomp/libconfig'

def pkgconfig_builder(env):
    pkgconfig = env.Substfile(
        'lib/pkgconfig/rotatable.pc',
        '#rotatable.pc.in',
        SUBST_DICT={'@prefix@': env['PREFIX']},
    )
    env.Alias(
        'install',
        env.Install(os.path.join(env['PREFIX'], 'lib/pkgconfig'), pkgconfig),
    )

def pkgconfig_parser(prefix):
    cmd = (
        'PKG_CONFIG_PATH=%s/lib/pkgconfig' % (prefix),
        'pkg-config',
        '--static',
        '--cflags',
        '--libs',
        'fsdyn',
    )
    return ' '.join(cmd)

def construct():
    ccflags = '-g -O2 -Wall -Wextra -Werror ' + os.getenv('FSCCFLAGS', '')
    linkflags = os.getenv('FSLINKFLAGS', '')
    ar_override = os.getenv('FSAR', os.getenv('FSBTAR', None))
    cc_override = os.getenv('FSCC', os.getenv('FSBTCC', None))
    ranlib_override = os.getenv('FSRANLIB', os.getenv('FSBTRANLIB', None))
    prefix = ARGUMENTS.get('prefix', '/usr/local')
    if ARGUMENTS.get('fscomp', 0):
        config_builder = libconfig_builder
        config_parser = libconfig_parser()
    else:
        config_builder = pkgconfig_builder
        config_parser = pkgconfig_parser(prefix)

    for target_arch in target_architectures():
        target_ccflags = TARGET_FLAGS[target_arch] + ccflags
        target_cppdefines = TARGET_DEFINES[target_arch]
        target_linkflags = TARGET_FLAGS[target_arch] + linkflags
        build_dir = os.path.join('stage',
                                 target_arch,
                                 ARGUMENTS.get('builddir', 'build'))
        for directory in DIRECTORIES:
            env = Environment(ARCH=target_arch,
                              CCFLAGS=target_ccflags,
                              CPPDEFINES=target_cppdefines,
                              LINKFLAGS=target_linkflags,
                              CONFIG_BUILDER=config_builder,
                              CONFIG_PARSER=config_parser,
                              PREFIX=prefix,
                              tools=['default', 'textfile'])
            env['ARCHBUILDDIR'] = env.Dir('#stage/$ARCH/build').abspath
            if ar_override:
                env['AR'] = ar_override
            if cc_override:
                env['CC'] = cc_override
            if ranlib_override:
                env['RANLIB'] = ranlib_override
            if target_arch == "darwin":
                env.AppendENVPath("PATH", "/opt/local/bin")
            SConscript(dirs=directory,
                       exports=['env'],
                       duplicate=False,
                       variant_dir=os.path.join(build_dir, directory))
        Clean('.', build_dir)

if __name__ == 'SCons.Script':
    construct()
