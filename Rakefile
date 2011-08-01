require 'rubygems'
require 'rbconfig'

USE_RAKE_COMPILER = (RUBY_PLATFORM =~ /java/) ? false : true
if USE_RAKE_COMPILER
  gem 'rake-compiler', '>=0.6.0'
  require 'rake/extensiontask'
  ENV['RUBY_CC_VERSION'] = '1.8.7:1.9.2'
end

require 'date'
require 'fileutils'
require 'rbconfig'

load 'tasks/setup.rb'

LIBEXT = case RbConfig::CONFIG['host_os'].downcase
  when /darwin/
    "dylib"
  when /mswin|mingw/
    "dll"
  else
    Config::CONFIG['DLEXT']
  end

CPU = case RbConfig::CONFIG['host_cpu'].downcase
  when /i[3456]86/
    # Darwin always reports i686, even when running in 64bit mode
    if Config::CONFIG['host_os'] =~ /darwin/ && 0xfee1deadbeef.is_a?(Fixnum)
      "x86_64"
    else
      "i386"
    end

  when /amd64|x86_64/
    "x86_64"

  when /ppc64|powerpc64/
    "powerpc64"

  when /ppc|powerpc/
    "powerpc"

  else
    Config::CONFIG['host_cpu']
  end

OS = case Config::CONFIG['host_os'].downcase
  when /linux/
    "linux"
  when /darwin/
    "darwin"
  when /freebsd/
    "freebsd"
  when /openbsd/
    "openbsd"
  when /sunos|solaris/
    "solaris"
  when /mswin|mingw/
    "win32"
  else
    Config::CONFIG['host_os'].downcase
  end

CC=ENV['CC'] || Config::CONFIG['CC'] || "gcc"

GMAKE = Config::CONFIG['host_os'].downcase =~ /bsd|solaris/ ? "gmake" : "make"

LIBTEST = "build/libtest.#{LIBEXT}"
BUILD_DIR = "build"
BUILD_EXT_DIR = File.join(BUILD_DIR, "#{Config::CONFIG['arch']}", 'ffi_c', RUBY_VERSION)

# Project general information
PROJ.name = 'ffi'
PROJ.authors = 'Wayne Meissner'
PROJ.email = 'wmeissner@gmail.com'
PROJ.url = 'http://wiki.github.com/ffi/ffi'
PROJ.version = '1.0.10'
PROJ.rubyforge.name = 'ffi'
PROJ.readme_file = 'README.rdoc'

# Annoucement
PROJ.ann.paragraphs << 'FEATURES' << 'SYNOPSIS' << 'REQUIREMENTS' << 'DOWNLOAD/INSTALL' << 'CREDITS' << 'LICENSE'

PROJ.ann.email[:from] = 'andrea.fazzi@alcacoop.it'
PROJ.ann.email[:to] = ['ruby-ffi@googlegroups.com']
PROJ.ann.email[:server] = 'smtp.gmail.com'

# Gem specifications
PROJ.gem.need_tar = false
PROJ.gem.files = %w(History.txt LICENSE README.rdoc Rakefile) + Dir.glob("{ext,gen,lib,spec,tasks}/**/*")
PROJ.gem.platform = Gem::Platform::RUBY
#PROJ.gem.required_ruby_version = ">= 1.9.2"

# Override Mr. Bones autogenerated extensions and force ours in
PROJ.gem.extras['extensions'] = %w(ext/ffi_c/extconf.rb)
#PROJ.gem.extras['required_ruby_version'] = ">= 1.9.2"

# RDoc
PROJ.rdoc.exclude << '^ext\/'
PROJ.rdoc.opts << '-x' << 'ext'

# Ruby
PROJ.ruby_opts = []
PROJ.ruby_opts << '-I' << BUILD_EXT_DIR unless RUBY_PLATFORM == "java"

# RSpec
PROJ.spec.files.exclude /rbx/
PROJ.spec.opts << '--color' << '-fs'

# Dependencies

#depend_on 'rake', '>=0.8.7'

TEST_DEPS = [ LIBTEST ]
if RUBY_PLATFORM == "java"
  desc "Run all specs"
  task :specs => TEST_DEPS do
    sh %{#{Gem.ruby} -S rspec #{Dir["spec/ffi/*_spec.rb"].join(" ")} -fs --color}
  end
  desc "Run rubinius specs"
  task :rbxspecs => TEST_DEPS do
    sh %{#{Gem.ruby} -S rspec #{Dir["spec/ffi/rbx/*_spec.rb"].join(" ")} -fs --color}
  end
else
  TEST_DEPS.unshift :compile
  desc "Run all specs"
  task :specs => TEST_DEPS do
    ENV["MRI_FFI"] = "1"
    sh %{#{Gem.ruby} -Ilib -I#{BUILD_EXT_DIR} -S rspec #{Dir["spec/ffi/*_spec.rb"].join(" ")} -fs --color}
  end
  desc "Run rubinius specs"
  task :rbxspecs => TEST_DEPS do
    ENV["MRI_FFI"] = "1"
    sh %{#{Gem.ruby} -Ilib -I#{BUILD_EXT_DIR} -S rspec #{Dir["spec/ffi/rbx/*_spec.rb"].join(" ")} -fs --color}
  end
end

desc "Build all packages"
task :package => 'gem:package'

desc "Install the gem locally"
task :install => 'gem:install'


desc "Clean all built files"
task :distclean => :clobber do
  FileUtils.rm_rf('build')
  FileUtils.rm_rf(Dir["lib/**/ffi_c.#{Config::CONFIG['DLEXT']}"])
  FileUtils.rm_rf('lib/1.8')
  FileUtils.rm_rf('lib/1.9')
  FileUtils.rm_rf('lib/ffi/types.conf')
  FileUtils.rm_rf('conftest.dSYM')
  FileUtils.rm_rf('pkg')
end


desc "Build the native test lib"
task "build/libtest.#{LIBEXT}" do
  sh %{#{GMAKE} -f libtest/GNUmakefile CPU=#{CPU} OS=#{OS} }
end


desc "Build test helper lib"
task :libtest => "build/libtest.#{LIBEXT}"

desc "Test the extension"
task :test => [ :specs, :rbxspecs ]


namespace :bench do
  ITER = ENV['ITER'] ? ENV['ITER'].to_i : 100000
  bench_libs = "-Ilib -I#{BUILD_DIR}" unless RUBY_PLATFORM == "java"
  bench_files = Dir["bench/bench_*.rb"].reject { |f| f == "bench_helper.rb" }
  bench_files.each do |bench|
    task File.basename(bench, ".rb")[6..-1] => TEST_DEPS do
      sh %{#{Gem.ruby} #{bench_libs} #{bench} #{ITER}}
    end
  end
  task :all => TEST_DEPS do
    bench_files.each do |bench|
      sh %{#{Gem.ruby} #{bench_libs} #{bench}}
    end
  end
end

task 'spec:run' => TEST_DEPS
task 'spec:specdoc' => TEST_DEPS

task :default => :specs

