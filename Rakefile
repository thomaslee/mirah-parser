# Copyright (c) 2010 The Mirah project authors. All Rights Reserved.
# All contributing project authors may be found in the NOTICE file.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

require 'ant'
require 'rake/testtask'

# load mirah rake task
$: << '../mirah/lib'
#require '/Developer/mirah-complete.jar'
require 'mirah_task'

task :default => :build_parser

task :clean do
  ant.delete :quiet => true, :dir => 'build'
  ant.delete :quiet => true, :dir => 'dist'
end

task :build_parser => 'dist/mirah-parser.jar'
ant.taskdef :name => 'jarjar', :classpath => 'javalib/jarjar-1.1.jar', :classname=>"com.tonicsystems.jarjar.JarJarTask"

file 'build/mirah-parser.jar' => ['build/mirahparser/lang/ast/Node.class',
                                  'build/mirahparser/impl/MirahParser.class',
                                  'build/mirahparser/impl/MirahLexer.class'] do
  ant.jarjar :jarfile => 'build/mirah-parser.jar' do
    fileset :dir => 'build', :includes => 'mirahparser/impl/*.class'
    fileset :dir => 'build', :includes => 'mirahparser/lang/ast/*.class'
    fileset :dir => 'build', :includes => 'org/mirahparser/ast/*.class'
    zipfileset :src => 'javalib/mmeta-runtime.jar'
    _element :rule, :pattern=>'mmeta.**', :result=>'org.mirahparser.mmeta.@1'
    manifest do
      attribute :name=>"Main-Class", :value=>"mirahparser.impl.MirahParser"
    end
  end
end

file 'dist/mirah-parser.jar' => 'build/mirah-parser.jar' do
  # Mirahc picks up the built in classes instead of our versions.
  # So we compile in a different package and then jarjar them to the correct
  # one.
  ant.jarjar :jarfile => 'dist/mirah-parser.jar' do
    zipfileset :src => 'build/mirah-parser.jar'
    _element :rule, :pattern=>'mirahparser.**', :result=>'mirah.@1'
    _element :rule, :pattern=>'org.mirahparser.**', :result=>'org.mirah.@1'
    manifest do
      attribute :name=>"Main-Class", :value=>"mirah.impl.MirahParser"
    end
  end
end

file 'build/mirahparser/impl/MirahParser.class' => ['build/mirahparser/impl/Mirah.mirah'] do
  mirahc('build/mirahparser/impl/Mirah.mirah',
         :dir => 'build',
         :dest => 'build',
         :options => ['--classpath', 'build:javalib/mmeta-runtime.jar'])
end

file 'build/org/mirahparser/ast/NodeMeta.class' => 'src/org/mirah/ast/meta.mirah' do
  mirahc('src/org/mirah/ast/meta.mirah',
         :dest => 'build'
         #:options => ['-V']
         )
end

file 'build/mirahparser/lang/ast/Node.class' =>
    ['build/org/mirahparser/ast/NodeMeta.class'] + Dir['src/mirah/lang/ast/*.mirah'] do
      mirahc('.',
             :dir => 'src/mirah/lang/ast',
             :dest => 'build',
             :options => ['--classpath', 'build'])
end

file 'build/mirahparser/lang/ast/Node.java' =>
    ['build/org/mirahparser/ast/NodeMeta.class'] + Dir['src/mirah/lang/ast/*.mirah'] do
      mirahc('.',
             :dir => 'src/mirah/lang/ast',
             :dest => 'build',
             :options => ['--java', '--classpath', 'build'])
end

file 'build/mirahparser/impl/MirahLexer.class' => Dir['src/mirahparser/impl/*.java'] do
  ant.javac :srcDir => 'src',
      :destDir => 'build',
      :debug => true do
    include :name => 'mirahparser/impl/Tokens.java'
    include :name => 'mirahparser/impl/MirahLexer.java'
    classpath :path => 'build:javalib/mmeta-runtime.jar'
  end
end

file 'build/mirahparser/impl/Mirah.mirah' => 'src/mirahparser/impl/Mirah.mmeta' do
  ant.mkdir :dir => 'build/mirahparser/impl'
  runjava 'javalib/mmeta.jar', '--tpl', 'node=src/mirahparser/impl/node.xtm', 'src/mirahparser/impl/Mirah.mmeta', 'build/mirahparser/impl/Mirah.mirah'
end

directory 'dist'
directory 'build/mirahparser/impl'

# TODO this uses the mirah parser from the compiler, not the version we
# just built.
Rake::TestTask.new :test do |t|
  # t.libs << 'build/test'
  t.test_files = FileList['test/*.rb']
end

task :test => 'build/mirah-parser.jar'

task :doc => 'build/mirahparser/lang/ast/Node.java' do
  ant.javadoc :sourcepath => 'build', :destdir => 'doc'
end

def runjava(jar, *args)
  options = {:failonerror => true, :fork => true}
  if jar =~ /\.jar$/
    options[:jar] = jar
  else
    options[:classname] = jar
  end
  options.merge!(args.pop) if args[-1].kind_of?(Hash)
  puts "java #{jar} " + args.join(' ')
  ant.java options do
    args.each do |value|
      arg :value => value
    end
  end
end