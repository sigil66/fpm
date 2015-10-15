NAME = 'fpm'
JRUBY = '1.7.19'
VERSION = '1.4.0'
EPOCH = VERSION.gsub(/[^0-9]/, '').gsub(/^0+/, '')
DESCRIPTION = 'Effing package management! Build packages for multiple platforms (deb, rpm, etc) with great ease and sanity.'
BUILD_OSX_DIR = './build_osx'
BUILD_LINUX_DIR = './build_linux'

ENV['PATH'] = Dir.pwd + "/jruby-#{JRUBY}/bin:#{ENV['PATH']}"

task :clean do
  sh %{ rm -rf #{BUILD_OSX_DIR} #{BUILD_LINUX_DIR} }
  sh %{ rm -rf ./jruby-* }
  sh %{ rm -rf ./fpm }
end

task :build do
  sh %{ curl --location -O http://jruby.org.s3.amazonaws.com/downloads/#{JRUBY}/jruby-bin-#{JRUBY}.zip }
  sh %{ unzip jruby-bin-#{JRUBY}.zip }

  sh %{ jruby -S gem install warbler fpm bundler rspec:3.0.0 insist:0.0.5 pry stud ffi json }
  sh %{ git clone --branch v#{VERSION} https://github.com/jordansissel/fpm }
  sh %{ cd fpm; warble jar }
end

task :layout_osx => [:build] do
  sh %{ mkdir -p #{BUILD_OSX_DIR}/opt/fpm/lib }
  sh %{ mkdir -p #{BUILD_OSX_DIR}/usr/local/bin }

  sh %{ cp fpm.sh #{BUILD_OSX_DIR}/usr/local/bin/fpm } 
  sh %{ chmod 755 #{BUILD_OSX_DIR}/usr/local/bin/fpm }
  sh %{ cp fpm/fpm.jar #{BUILD_OSX_DIR}/opt/fpm/lib/ }
end

task :layout_linux => [:build] do
  sh %{ mkdir -p #{BUILD_LINUX_DIR}/opt/fpm/lib }
  sh %{ mkdir -p #{BUILD_LINUX_DIR}/usr/bin }

  sh %{ cp fpm.sh #{BUILD_LINUX_DIR}/usr/bin/fpm } 
  sh %{ chmod 755 #{BUILD_LINUX_DIR}/usr/bin/fpm }
  sh %{ cp fpm/fpm.jar #{BUILD_LINUX_DIR}/opt/fpm/lib/ }
end

task :osxpkg => [:layout_osx] do
  sh %{ java -jar #{BUILD_OSX_DIR}/opt/fpm/lib/fpm.jar \
              -t osxpkg \
              -s dir \
              --osxpkg-identifier-prefix com.jen20 \
              -n #{NAME} \
              -v #{VERSION} \
              -a all \
              -C #{BUILD_OSX_DIR} \
              .
  }
end

task :deb => [:layout_linux] do
  sh %{ java -jar #{BUILD_LINUX_DIR}/opt/fpm/lib/fpm.jar \
              -t deb \
              -s dir \
              -n #{NAME} \
              -v #{VERSION} \
              --description "#{DESCRIPTION}" \
              -a all \
              --depends "java-runtime-headless" \
              --deb-no-default-config-files \
              --deb-user root \
              --deb-group root \
              -C #{BUILD_LINUX_DIR} \
              .
  }
end

task :rpm => [:layout_linux] do
  sh %{ java -jar #{BUILD_LINUX_DIR}/opt/fpm/lib/fpm.jar \
              -t rpm \
              -s dir \
              -n #{NAME} \
              -v #{VERSION} \
              --description "#{DESCRIPTION}" \
              --epoch #{EPOCH} \
              -a all \
              --rpm-os linux \
              --rpm-user root \
              --rpm-group root \
              --rpm-use-file-permissions \
              --depends "jre >= 1.7" \
              -C #{BUILD_LINUX_DIR} \
              .
  }
end

task :default => [ :clean, :build, :layout_osx, :osxpkg, :layout_linux, :deb, :rpm ]
