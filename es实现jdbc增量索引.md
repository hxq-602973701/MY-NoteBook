#### linux上安装 logstash6.3.1 以及logstash-input-jdbc4.3.9 实现增量索引

* 首先安装logstash6.3.1 和 logstash-input-jdbc4.3.9
  * 测试有没有安装成功
  ```sh
	cd bin
	./logstash -e 'input { stdin { } } output { stdout {} }'
 * 然后
 ```sh
  gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
  * 然后
   ```sh
  gem sources -l
  * 然后进入logstash-6.3.1，修改Gemfile文件里面的数据源
 ```sh
  vi Gemfile
  source "https://gems.ruby-china.org"
 * 然后进入 logstash-input-jdbc4.3.9 下
 ```sh
  gem build logstash-input-jdbc.gemspec
  * 然后进入logstash-6.3.1/bin 下
  ```sh
  bin/plugin install /your/local/plugin/logstash-input-jdbc.gem
  * 然后看是否成功 如果成功了  在bin目录下创建一个conf文件内容如下:
  ```sh
  input {
	  jdbc {
	    jdbc_driver_library => "/path/to/mysql-connector-java-5.1.33-bin.jar"
	    jdbc_driver_class => "com.mysql.jdbc.Driver"
	    jdbc_connection_string => "jdbc:mysql://host:port/database"
	    jdbc_user => "user"
	    jdbc_password => "password"
      # or jdbc_password_filepath => "/path/to/my/password_file"
	    statement => "SELECT ..."
	    jdbc_paging_enabled => "true"
	    jdbc_page_size => "50000"
	  }
	}

	filter {
	  [some filters here]
	}

	output {
	  stdout {
	    codec => rubydebug
	  }
	  elasticsearch_http {
	    host => "host"
	    index => "myindex"
	  }
	}
  * 然后启动就可以测试了
  ```sh
  ./logstash -f config-mysql/mysql.conf 
  
    * [linux上安装 logstash6.3.1 以及logstash-input-jdbc4.3.9 实现增量索引](https://blog.csdn.net/q15150676766/article/details/75949679)
  
