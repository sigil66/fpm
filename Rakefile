NAME = 'fpm'
JRUBY = '1.7.19'
VERSION = '1.3.3'
BUILD_DIR = './build'

ENV['PATH'] = Dir.pwd + "/jruby-#{JRUBY}/bin:#{ENV['PATH']}"

task :clean do
  sh %{ rm -rf #{BUILD_DIR} }
  sh %{ rm -rf ./jruby-* }
  sh %{ rm -rf ./fpm }
end

task :build do
  sh %{ mkdir -p #{BUILD_DIR}/opt/fpm/lib }
  sh %{ mkdir -p #{BUILD_DIR}/usr/bin }

  sh %{ cp fpm.sh #{BUILD_DIR}/usr/bin/fpm } 
  sh %{ chmod 755 #{BUILD_DIR}/usr/bin/fpm }

  sh %{ wget http://jruby.org.s3.amazonaws.com/downloads/#{JRUBY}/jruby-bin-#{JRUBY}.zip }
  sh %{ unzip jruby-bin-#{JRUBY}.zip }

  sh %{ jruby -S gem install warbler fpm bundler rspec:3.0.0 insist:0.0.5 pry stud ffi json }
  sh %{ git clone --branch v#{VERSION} https://github.com/jordansissel/fpm }
  sh %{ cd fpm; warble jar }
  sh %{ mv fpm/fpm.jar #{BUILD_DIR}/opt/fpm/lib/ }
end

task :deb do
  sh %{ java -jar #{BUILD_DIR}/opt/fpm/lib/fpm.jar -t deb -s dir -n #{NAME} -v #{VERSION} -a all -d "java-runtime-headless" --deb-no-default-config-files --deb-user root --deb-group root -C ./build . }
end

task :default => [ :clean, :build, :deb ]
