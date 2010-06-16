# -*- python -*-

import os
import sys
import platform

# Home made tests
sys.path.append('SconsTests')
import wxconfig
import utils

# Some features need at least SCons 1.2
EnsureSConsVersion(1, 2)

warnings = [
    'all',
    'write-strings',
    'shadow',
    'pointer-arith',
    'packed',
    'no-conversion',
    ]

compileFlags = [
    '-fno-exceptions',
    '-fno-strict-aliasing',
    '-msse2',
    '-fPIC',
    ]

cppDefines = [
    ( '_FILE_OFFSET_BITS', 64),
    '_LARGEFILE_SOURCE',
    'GCC_HASCLASSVISIBILITY',
    ]

basedir = os.getcwd() + '/'

include_paths = [
    basedir + 'Source/Core/Common/Src',
    basedir + 'Source/Core/DiscIO/Src',
    basedir + 'Source/PluginSpecs',
    basedir + 'Source/Core/Core/Src',
    basedir + 'Source/Core/DebuggerWX/Src',
    basedir + 'Externals/Bochs_disasm',
    basedir + 'Externals/Lua',
    basedir + 'Externals/WiiUseSrc/Src',
    basedir + 'Source/Core/VideoCommon/Src',
    basedir + 'Source/Core/InputCommon/Src',
    basedir + 'Source/Core/InputUICommon/Src',
    basedir + 'Source/Core/AudioCommon/Src',
    basedir + 'Source/Core/DebuggerUICommon/Src',
    basedir + 'Source/Core/DolphinWX/Src',
    basedir + 'Source/Core/DSPCore/Src',
    ]

dirs = [
    'Externals/Bochs_disasm',
    'Externals/Lua',
    'Externals/MemcardManager',
    'Externals/WiiUseSrc/Src',
    'Source/Core/Common/Src',
    'Source/Core/Core/Src',
    'Source/Core/DiscIO/Src',
    'Source/Core/VideoCommon/Src',
    'Source/Core/InputCommon/Src',
    'Source/Core/AudioCommon/Src',
    'Source/Core/DebuggerUICommon/Src',
    'Source/Core/DSPCore/Src',
    'Source/DSPTool/Src',
    'Source/Core/InputUICommon/Src/',
    'Source/Plugins/Plugin_VideoOGL/Src',
    'Source/Plugins/Plugin_VideoSoftware/Src',
    'Source/Plugins/Plugin_DSP_HLE/Src',
    'Source/Plugins/Plugin_DSP_LLE/Src',
    'Source/Plugins/Plugin_Wiimote/Src',
    'Source/Plugins/Plugin_WiimoteNew/Src/',
    'Source/Core/DolphinWX/Src',
    'Source/Core/DebuggerWX/Src',
    'Source/UnitTests/',
    ]

builders = {}
if sys.platform == 'darwin':
    from plistlib import writePlist
    def createPlist(target, source, env):
        properties = {}
        for srcNode in source:
            properties.update(srcNode.value)
            for dstNode in target:
                writePlist(properties, str(dstNode))
    builders['Plist'] = Builder(action = createPlist)

# Handle command line options
vars = Variables('args.cache')

vars.AddVariables(
    BoolVariable('verbose', 'Set for compilation line', False),
    BoolVariable('bundle', 'Set to create bundle', False),
    BoolVariable('lint', 'Set for lint build (extra warnings)', False),
    BoolVariable('nowx', 'Set for building with no WX libs', False),
    BoolVariable('wxgl', 'Set for building with WX GL on Linux', False),
    BoolVariable('opencl', 'Build with OpenCL', False),
    BoolVariable('nojit', 'Remove entire jit cores', False),
    PathVariable('userdir', 'Set the name of the user data directory in home',
                 '.dolphin-emu', PathVariable.PathAccept),
    EnumVariable('install', 'Choose a local or global installation', 'local',
                 allowed_values = ('local', 'global'),
                 ignorecase = 2
                 ),
    PathVariable('prefix', 'Installation prefix (only used for a global build)',
                 '/usr', PathVariable.PathAccept),
    PathVariable('destdir', 'Temporary install location (for package building)',
                 None, PathVariable.PathAccept),
    EnumVariable('flavor', 'Choose a build flavor', 'release',
                 allowed_values = ('release','devel','debug','fastlog','prof'),
                 ignorecase = 2
                 ),
    PathVariable('wxconfig', 'Path to the wxconfig', None),
    EnumVariable('pgo', 'Profile-Guided Optimization (generate or use)', 'none',
                allowed_values = ('none', 'generate', 'use'),
                ignorecase = 2
                ),
    ('CC', 'The c compiler', 'gcc'),
    ('CXX', 'The c++ compiler', 'g++'),
    )

env = Environment(
    CPPPATH = include_paths,
    RPATH = [],
    LIBS = [],
    LIBPATH = [],
    BUILDERS = builders,
    variables = vars,
    )

if sys.platform == 'win32':
    env['tools'] = ['mingw']
    env['ENV'] = os.environ
else:
    env['ENV'] = {
        'PATH' : os.environ['PATH'],
        'HOME' : os.environ['HOME'],
        'PKG_CONFIG_PATH' : os.environ.get('PKG_CONFIG_PATH')
    }

# Save the given command line options
vars.Save('args.cache', env)

# Verbose compile
if not env['verbose']:
    env['CCCOMSTR'] = "Compiling $TARGET"
    env['CXXCOMSTR'] = "Compiling $TARGET"
    env['ARCOMSTR'] = "Archiving $TARGET"
    env['LINKCOMSTR'] = "Linking $TARGET"
    env['ASCOMSTR'] = "Assembling $TARGET"
    env['ASPPCOMSTR'] = "Assembling $TARGET"
    env['SHCCCOMSTR'] = "Compiling shared $TARGET"
    env['SHCXXCOMSTR'] = "Compiling shared $TARGET"
    env['SHLINKCOMSTR'] = "Linking shared $TARGET"
    env['RANLIBCOMSTR'] = "Indexing $TARGET"

# Build flavor
flavour = env['flavor']
if (flavour == 'debug'):
    compileFlags.append('-ggdb')
    cppDefines.append('_DEBUG') #enables LOGGING
    # FIXME: this disable wx debugging how do we make it work?
    cppDefines.append('NDEBUG')
elif (flavour == 'devel'):
    compileFlags.append('-ggdb')
elif (flavour == 'fastlog'):
    compileFlags.append('-O3')
    cppDefines.append('DEBUGFAST')
elif (flavour == 'prof'):
    compileFlags.append('-O3')
    compileFlags.append('-ggdb')
elif (flavour == 'release'):
    compileFlags.append('-O3')
    compileFlags.append('-fomit-frame-pointer');
# More warnings
if env['lint']:
    warnings.append('error')
    # Should check for the availability of these (in GCC 4.3 or newer)
    if sys.platform != 'darwin':
        warnings.append('no-array-bounds')
        warnings.append('no-unused-result')
    # wxWidgets causes too many warnings with these
    #warnings.append('unreachable-code')
    #warnings.append('float-equal')

# Add the warnings to the compile flags
compileFlags += [ ('-W' + warning) for warning in warnings ]

env['CCFLAGS'] = compileFlags
if sys.platform == 'win32':
    env['CXXFLAGS'] = compileFlags
else:
    env['CXXFLAGS'] = compileFlags + [ '-fvisibility-inlines-hidden' ]
env['CPPDEFINES'] = cppDefines

# PGO - Profile Guided Optimization
if env['pgo']=='generate':
    compileFlags.append('-fprofile-generate')
    env['LINKFLAGS']='-fprofile-generate'
if env['pgo']=='use':
    compileFlags.append('-fprofile-use')
    env['LINKFLAGS']='-fprofile-use'

# Configuration tests section
tests = {'CheckWXConfig' : wxconfig.CheckWXConfig,
         'CheckPKGConfig' : utils.CheckPKGConfig,
         'CheckPKG' : utils.CheckPKG,
         'CheckSDL' : utils.CheckSDL,
         'CheckPortaudio' : utils.CheckPortaudio,
         }

# Object files
env['build_dir'] = os.path.join(basedir, 'Build',
    platform.system() + '-' + platform.machine() + '-' + env['flavor'] + os.sep)

# Static libs go here
env['local_libs'] = env['build_dir'] + os.sep + 'libs' + os.sep

# Where do we run from
env['base_dir'] = os.getcwd()+ '/'

# Install paths
extra=''
if flavour == 'debug':
    extra = '-debug'
elif flavour == 'prof':
    extra = '-prof'

# Set up the install locations
if (env['install'] == 'global'):
    env['prefix'] = os.path.join(env['prefix'] + os.sep)
    env['binary_dir'] = env['prefix'] + 'bin/'
    env['plugin_dir'] = env['prefix'] + 'lib/dolphin-emu/'
    env['data_dir'] = env['prefix'] + "share/dolphin-emu/"
else:
    env['prefix'] = os.path.join(env['base_dir'] + 'Binary',
        platform.system() + '-' + platform.machine() + extra + os.sep)
    env['binary_dir'] = env['prefix']
    env['plugin_dir'] = env['prefix'] + 'plugins/'
    env['data_dir'] = env['prefix']
if sys.platform == 'darwin':
    env['plugin_dir'] = env['prefix'] + 'Dolphin.app/Contents/PlugIns/'
    env['data_dir'] = env['prefix'] + 'Dolphin.app/Contents/'

conf = env.Configure(custom_tests = tests,
                     config_h="Source/Core/Common/Src/Config.h")

env['HAVE_OPENCL'] = 0
if env['opencl']:
    env['HAVE_OPENCL'] = conf.CheckPKG('OpenCL')

# OS X specifics
if sys.platform == 'darwin':
    # OpenCL is new in OS X 10.6. Other than OpenCL,
    # we try to maintain 10.5 compatibility, however.
    if not env['HAVE_OPENCL']:
        compileFlags += ['-isysroot', '/Developer/SDKs/MacOSX10.5.sdk']
    compileFlags += ['-mmacosx-version-min=10.5']
    conf.Define('MAP_32BIT', 0)
    env['CC'] = "gcc-4.2"
    env['CFLAGS'] = ['-x', 'objective-c']
    env['CXX'] = "g++-4.2"
    env['CXXFLAGS'] = ['-x', 'objective-c++']
    env['CCFLAGS'] += ['-arch' , 'x86_64' , '-arch' , 'i386']
    env['LINKFLAGS'] += ['-arch', 'x86_64' , '-arch' , 'i386']
    env['FRAMEWORKS'] += ['CoreFoundation', 'CoreServices']
    env['FRAMEWORKS'] += ['IOBluetooth', 'IOKit', 'OpenGL']
    env['FRAMEWORKS'] += ['AudioUnit', 'CoreAudio']
else:
    if not conf.CheckPKGConfig('0.15.0'):
        print "Can't find pkg-config, some tests will fail"

shared = {}
shared['glew'] = shared['lzo'] = shared['sdl'] = \
shared['soil'] = shared['sfml'] = shared['zlib'] = 0
if not sys.platform == 'darwin':
    # GLEW, SDL and zlib are present on almost all Linux systems,
    # so it is generally safe to link with their shared distribution
    # libraries even when we are building a redistributable image.
    shared['glew'] = conf.CheckPKG('GLEW')
    shared['sdl'] = conf.CheckPKG('SDL')
    shared['zlib'] = conf.CheckPKG('z')
    # LZO, SFML and SOIL are not typically part of a default install
    # of most Linux distributions, so only link with any of these
    # shared libraries if we are just doing a development build.
    if flavour != 'release':
        shared['lzo'] = conf.CheckPKG('lzo2')
        # TODO:  Check the version of sfml.  It should be at least version 1.5
        shared['sfml'] = conf.CheckPKG('sfml-network') and \
                         conf.CheckCXXHeader("SFML/Network/Ftp.hpp")
        shared['soil'] = conf.CheckPKG('SOIL')
    for lib in shared:
        if not shared[lib]:
            print "Shared library " + lib + " not detected, " \
                  "falling back to the static library"

if shared['glew'] == 0:
    env['CPPPATH'] += [basedir + 'Externals/GLew/include']
    dirs += ['Externals/GLew']
if shared['lzo'] == 0:
    env['CPPPATH'] += [basedir + 'Externals/LZO']
    dirs += ['Externals/LZO']
if shared['sdl'] == 0:
    env['CPPPATH'] += [basedir + 'Externals/SDL']
    env['CPPPATH'] += [basedir + 'Externals/SDL/include']
    dirs += ['Externals/SDL']
if shared['soil'] == 0:
    env['CPPPATH'] += [basedir + 'Externals/SOIL']
    dirs += ['Externals/SOIL']
if shared['sfml'] == 0:
    env['CPPPATH'] += [basedir + 'Externals/SFML/include']
    dirs += ['Externals/SFML/src']
if shared['zlib'] == 0:
    env['CPPPATH'] += [basedir + 'Externals/zlib']
    dirs += ['Externals/zlib']

wxmods = ['aui', 'adv', 'core', 'base']
if env['wxgl'] or sys.platform == 'win32' or sys.platform == 'darwin':
    env['USE_WX'] = 1
    wxmods.append('gl')
else:
    env['USE_WX'] = 0;
if env['nowx']:
    env['USE_WX'] = 0;

if sys.platform == 'darwin':
    wxver = '2.9' # 64-bit on OS X
else:
    wxver = '2.8'

if env['nowx']:
    env['HAVE_WX'] = 0;
else:
    env['HAVE_WX'] = conf.CheckWXConfig(wxver, wxmods, 0)
    wxconfig.ParseWXConfig(env)
    # wx-config wants us to link with the OS X QuickTime framework
    # which is not available for x86_64 and we don't use it anyway.
    # Strip it out to silence some harmless linker warnings.
    if env['FRAMEWORKS'].count('QuickTime'):
        env['FRAMEWORKS'].remove('QuickTime')
    # Make sure that the libraries claimed by wx-config are valid
    env['HAVE_WX'] = conf.CheckPKG('c')

if not env['HAVE_WX'] and not env['nowx']:
    print "WX libraries not found - see config.log"
    Exit(1)

if not sys.platform == 'win32':
    if not conf.CheckPKG('Cg'):
        print "Must have Cg framework from NVidia to build"
        Exit(1)

env['HAVE_BLUEZ'] = 0
env['HAVE_ALSA'] = env['HAVE_AO'] = env['HAVE_OPENAL'] = \
env['HAVE_PORTAUDIO'] = env['HAVE_PULSEAUDIO'] = 0
env['HAVE_X11'] = env['HAVE_XRANDR'] = 0
if not sys.platform == 'win32' and not sys.platform == 'darwin':
    env['LINKFLAGS'] += ['-pthread']

    env['HAVE_BLUEZ'] = conf.CheckPKG('bluez')

    env['HAVE_ALSA'] = conf.CheckPKG('alsa')
    env['HAVE_AO'] = conf.CheckPKG('ao')
    env['HAVE_OPENAL'] = conf.CheckPKG('openal')
    env['HAVE_PORTAUDIO'] =  conf.CheckPortaudio(1890)
    env['HAVE_PULSEAUDIO'] = conf.CheckPKG('libpulse-simple')

    env['HAVE_X11'] = conf.CheckPKG('x11')
    env['HAVE_XRANDR'] = env['HAVE_X11'] and conf.CheckPKG('xrandr')
    if not conf.CheckPKG('GL'):
        print "Must have OpenGL to build"
        Exit(1)
    if not conf.CheckPKG('GLU'):
        print "Must have GLU to build"
        Exit(1)
    if not conf.CheckPKG('CgGL'):
        print "Must have CgGl to build"
        Exit(1)

# Check for GTK 2.0 or newer
if sys.platform == 'linux2':
    if env['HAVE_WX'] and not conf.CheckPKG('gtk+-2.0'):
        print "gtk+-2.0 developement headers not detected"
        print "gtk+-2.0 is required to build the WX GUI"
        Exit(1)

env['NOJIT'] = 0
if env['nojit']:
    env['NOJIT'] = 1

conf.Define('NOJIT', env['NOJIT'])

# Creating config.h defines
conf.Define('HAVE_BLUEZ', env['HAVE_BLUEZ'])
conf.Define('HAVE_AO', env['HAVE_AO'])
conf.Define('HAVE_OPENCL', env['HAVE_OPENCL'])
conf.Define('HAVE_OPENAL', env['HAVE_OPENAL'])
conf.Define('HAVE_ALSA', env['HAVE_ALSA'])
conf.Define('HAVE_PULSEAUDIO', env['HAVE_PULSEAUDIO'])
conf.Define('HAVE_WX', env['HAVE_WX'])
conf.Define('USE_WX', env['USE_WX'])
conf.Define('HAVE_X11', env['HAVE_X11'])
conf.Define('HAVE_XRANDR', env['HAVE_XRANDR'])
conf.Define('HAVE_PORTAUDIO', env['HAVE_PORTAUDIO'])
conf.Define('USER_DIR', "\"" + env['userdir'] + "\"")
if (env['install'] == 'global'):
    conf.Define('DATA_DIR', "\"" + env['data_dir'] + "\"")
    conf.Define('LIBS_DIR', "\"" + env['prefix'] + 'lib/' +  "\"")

# Lua
env['LUA_USE_MACOSX'] = env['LUA_USE_LINUX'] = env['LUA_USE_POSIX'] = 0
if sys.platform == 'darwin':
    env['LUA_USE_MACOSX'] = 1
elif sys.platform == 'linux2':
    env['LUA_USE_LINUX'] = 1

conf.Define('LUA_USE_MACOSX', env['LUA_USE_MACOSX'])
conf.Define('LUA_USE_LINUX', env['LUA_USE_LINUX'])

# Profiling
env['USE_OPROFILE'] = 0
if (flavour == 'prof'):
    proflibs = [ '/usr/lib/oprofile', '/usr/local/lib/oprofile' ]
    env['LIBPATH'].append(proflibs)
    env['RPATH'].append(proflibs)
    if conf.CheckPKG('opagent'):
        env['USE_OPROFILE'] = 1
    else:
        print "Can't build prof without oprofile, disabling"

conf.Define('USE_OPROFILE', env['USE_OPROFILE'])
# After all configuration tests are done
conf.Finish()

env['LIBPATH'].append(env['local_libs'])

rev = utils.GenerateRevFile(env['flavor'],
                            "Source/Core/Common/Src/svnrev_template.h",
                            "Source/Core/Common/Src/svnrev.h")
# Print a nice progress indication when not compiling
Progress(['-\r', '\\\r', '|\r', '/\r'], interval=5)

# Setup destdir for package building
# Warning:  The program will not run from this location.  It is assumed the
# package will later install it to the prefix as it was defined before this.
if env.has_key('destdir'):
    env['prefix'] = env['destdir'] + env['prefix']
    env['plugin_dir'] = env['destdir'] + env['plugin_dir']
    env['binary_dir'] = env['destdir'] + env['binary_dir']
    env['data_dir'] = env['destdir'] + env['data_dir']

# Die on unknown variables
unknown = vars.UnknownVariables()
if unknown:
    print "Unknown variables:", unknown.keys()
    Exit(1)

# Generate help
Help(vars.GenerateHelpText(env))

Export('env')

for subdir in dirs:
    SConscript(
        subdir + os.sep + 'SConscript',
        variant_dir = env[ 'build_dir' ] + subdir + os.sep,
        duplicate=0
        )

# Data install
if sys.platform == 'darwin':
    env.Install(env['data_dir'], 'Data/Sys')
    env.Install(env['data_dir'], 'Data/User')
    env.Install(env['binary_dir'] + 'Dolphin.app/Contents/Resources/',
                'Source/Core/DolphinWX/resources/Dolphin.icns')
else:
    env.InstallAs(env['data_dir'] + 'sys', 'Data/Sys')
    env.InstallAs(env['data_dir'] + 'user', 'Data/User')

env.Alias('install', env['prefix'])

if env['bundle']:
    if sys.platform == 'linux2':
        # Make tar ball (TODO put inside normal dir)
        tar_env = env.Clone()
        tarball = tar_env.Tar('dolphin-'+rev +'.tar.bz2', env['prefix'])
        tar_env.Append(TARFLAGS='-j', TARCOMSTR="Creating release tarball")
    elif sys.platform == 'darwin':
        env.Command('.', env['binary_dir'] +
                    'Dolphin.app/Contents/MacOS/Dolphin', './osx_make_dmg.sh')

#TODO clean all bundles
#env.Clean(all, 'dolphin-*' + '.tar.bz2')
#env.Clean(all, 'Binary/Dolphin-r*' + '.dmg')
