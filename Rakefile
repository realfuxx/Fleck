require 'rubygems'
require 'albacore'

load 'support/platform.rb'

ROOT_NAMESPACE = 'Fleck'
PRODUCT = ROOT_NAMESPACE
COPYRIGHT = 'Copyright Jason Staten 2010-2011. All rights reserved.'
DESCRIPTION = 'C# WebSocket Implementation'
COMMON_ASSEMBLY_INFO = 'src/CommonAssemblyInfo.cs'
COMPILE_TARGET = ENV['COMPILE_TARGET'] || 'Release'
COMPILE_PLATFORM = 'Any CPU'
CLR_TOOLS_VERSION = 'v4.0.30319'
BUILD_RUNNER = Platform.nix? ? 'xbuild' : 'msbuild'
ARCHIVE_DIR = 'artifacts'
RESULTS_DIR = 'artifacts/test-reports'

desc 'Compiles and runs unit tests'
task :all => [:default]

desc 'Compiles and runs tests'
task :default => [:build, :test]

desc 'Build application'
task :build => [:clean, :version, :compile] do
  copyOutputFiles "src/#{ROOT_NAMESPACE}/bin/#{COMPILE_TARGET}", "*.{dll,pdb}", ARCHIVE_DIR
end

desc 'Update the version information for the build'
assemblyinfo :version do |asm|
  tc_build_number = ENV['BUILD_NUMBER']
  build_revision = tc_build_number || Time.new.strftime('5%H%M')

  description = `git describe --long`.chomp
  segments = description.split '-'
  asm_version = "#{segments[0]}.#{segments[1]}"

  commit = `git log -1 --pretty=format:%H`

  asm.trademark = commit
  asm.product_name = PRODUCT
  asm.description = DESCRIPTION
  asm.version = asm_version
  asm.file_version = asm_version
  asm.custom_attributes :AssemblyInformationalVersion => asm_version
  asm.copyright = COPYRIGHT
  asm.output_file = COMMON_ASSEMBLY_INFO
end

desc 'Prepares the working directory for a new build'
task :clean do
  Rake::Task["clean:#{BUILD_RUNNER}"].execute
  rm_r ARCHIVE_DIR if Dir.exists?(ARCHIVE_DIR)
  rm_r RESULTS_DIR if Dir.exists?(RESULTS_DIR)
  mkdir_p ARCHIVE_DIR
  mkdir_p RESULTS_DIR
end

namespace :clean do
  msbuild :msbuild do |msb|
    clean_solution(msb)
  end
  xbuild :xbuild do |xb|
    clean_solution(xb)
  end

  def clean_solution(command)
    command.targets :Clean
    command.solution = "src/#{ROOT_NAMESPACE}.sln"
    command.properties = {
      :configuration => COMPILE_TARGET,
      :platform => COMPILE_PLATFORM
    }
  end
end

task :compile do
  Rake::Task["compile:#{BUILD_RUNNER}"].execute
end

namespace :compile do
  msbuild :msbuild do |msb|
    compile_solution(msb)
  end
  xbuild :xbuild do |xb|
    compile_solution(xb)
  end

  def compile_solution(command)
    command.solution = "src/#{ROOT_NAMESPACE}.sln"
    command.properties = {
      :configuration => COMPILE_TARGET,
      :platform => COMPILE_PLATFORM
    }
  end
end

desc 'Build the nuget package'
task :nuget => [:build] do
  sh "nuget pack -OutputDirectory #{ARCHIVE_DIR} -Properties Configuration=#{COMPILE_TARGET} src/#{ROOT_NAMESPACE}/#{ROOT_NAMESPACE}.csproj"
end

desc 'Run tests'
task :test do
  runner = Dir['**/nunit-console.exe'].first
  raise "nunit-console.exe not found" if runner.nil?
  assemblies = Dir["**/#{COMPILE_TARGET}/*.Tests.dll"].reject{|a|a =~ /\/obj\//}
  output = File.join(RESULTS_DIR, 'TestResults.xml')

  sh "#{Platform.runtime(runner)} #{assemblies.join} #{Platform.switch('xml:')}#{output}"
end

def copyOutputFiles(fromDir, filePattern, outDir)
  copy Dir[File.join(fromDir, filePattern)], outDir
end
