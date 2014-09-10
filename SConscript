# -*- python -*-

#
# SConscript file to specify the build process, see:
# http://scons.org/doc/production/HTML/scons-man.html
#
Decider('MD5')
import platform
import fnmatch
import os

vars = Variables()
vars.Add(BoolVariable('DEBUG', 'Set to disable optimizations', False))
vars.Add(BoolVariable('PIC', 'Set to 1 for to always generate PIC code', True))
env = Environment(variables = vars)

# Augmate tweaks:
#
# create your specialized toolchain:
# bash ~/android-ndk/build/tools/make-standalone-toolchain.sh --platform=android-19 --install-dir=/tmp/android-toolchain-x86 --arch=x86
# bash ~/android-ndk/build/tools/make-standalone-toolchain.sh --platform=android-19 --install-dir=/tmp/android-toolchain-arm --arch=arm
#
# use --arm and --x86 options to toggle between builds

AddOption('--platform', dest='platform', type='string', nargs=1, action='store', default='x86', help='target platform (valid values: "x86" or "arm")')
target_platform = GetOption('platform')
print "Building for platform = " + target_platform

compile_options = {}

# Force ANSI (C++98) to ensure compatibility with MSVC.
cxxflags = ['-ansi -pedantic -fPIC -fexceptions']
if env['DEBUG']:
  #compile_options['CPPDEFINES'] = '-DDEBUG'
  cxxflags.append('-O0 -g3 -ggdb -DNO_ICONV')
  cxxflags.append('-Wall -Wextra')
  # -Werror
else:
  cxxflags.append('-Os -Wall -Wextra -DNO_ICONV')
compile_options['CXXFLAGS'] = ' '.join(cxxflags)

# platform specific flags and paths
if target_platform == "x86":
  platform_folder = '/tmp/android-toolchain-x86/bin/i686-linux-android-'
  compile_options['LINKFLAGS'] = '-ldl -lstdc++ -L/tmp/android-toolchain-x86/i686-linux-android/lib'
else:
  platform_folder = '/tmp/android-toolchain-arm/bin/arm-linux-androideabi-'
  cxxflags.append('-march=arm7v-a -mfloat-abi=softfp -mfpu=neon')
  compile_options['LINKFLAGS'] = '-ldl -lstdc++ -L/tmp/android-toolchain-arm/arm-linux-android/lib'

# gcc toolchain bin paths
env.Replace(CXX = platform_folder + 'g++')
env.Replace(CC = platform_folder + 'gcc')
env.Replace(AR = platform_folder + 'ar')
env.Replace(RANLIB = platform_folder + 'ranlib')
env.Replace(LINK = platform_folder + 'g++')
env.Replace(LD = platform_folder + 'ld')

def all_files(dir, ext='.cpp', level=6):
  files = []
  for i in range(1, level):
    files += Glob(dir + ('/*' * i) + ext)
  return files

def all_libs(name, dir):
  matches = []
  for root, dirnames, filenames in os.walk(dir):
    for filename in fnmatch.filter(filenames, name):
      matches.append(os.path.join(root, filename))
  return matches

# Setup libiconv, if possible
libiconv_include = []
libiconv_libs = []
if all_libs('libiconv.*', '/opt/local/lib'):
  libiconv_include.append('/opt/local/include/')
  libiconv_libs.append('iconv')
else:
  if all_libs('libiconv.*', '/usr/lib'):
    libiconv_libs.append('iconv')

# Add libzxing library.
libzxing_files = all_files('core/src')+all_files('core/src', '.cc')
libzxing_include = ['core/src']
if platform.system() is 'Windows':
  libzxing_files += all_files('core/src/win32')
  libzxing_include += ['core/src/win32']
libzxing = env.SharedLibrary('zxing', source=libzxing_files,
  CPPPATH=libzxing_include + libiconv_libs, **compile_options)

# Add cli.
zxing_files = all_files('cli/src')
zxing = env.Program('zxing', zxing_files,
  CPPPATH=libzxing_include,
  LIBS=libzxing + libiconv_libs, **compile_options)

# Setup CPPUnit.
cppunit_include = ['/opt/local/include/']
cppunit_libs = ['cppunit']

# Add testrunner program.
test_files = all_files('core/tests/src')
test = env.Program('testrunner', test_files,
  CPPPATH=libzxing_include + cppunit_include,
  LIBS=libzxing + cppunit_libs, **compile_options)

# Setup some aliases.
Alias('lib', libzxing)
Alias('zxing', zxing)
Alias('tests', test)
