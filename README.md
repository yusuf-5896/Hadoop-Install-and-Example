# Hadoop 2.7.2 Multi-Node Kurulumu Ve WordCount Örneği

## Gereksinimler:
	* Ubuntu 18.04 LTS
	* Hadoop-2.7.2
	* JAVA 8
	* SSH


## 1. Adım

İlk önce bilgisayarlarda **"hadoop"** isimli yönetici olarak ve şifreleri aynı olmak üzere kullanıcı oluşturalım. Ve bu kullanıcı ile oturum açalım.

![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_user.png)


## 2.adım
şimdi java ile ssh kuralım. Terminalde aşağıdaki komutları sırası ile yazıyoruz.

	sudo add-apt-repository ppa:webupd8team/java
	sudo apt-get update
	sudo apt-get install oracle-java8-installer
	sudo apt-get install openssh-server openssh-client

## 3.adım
**"/ect/hosts"** dosyasını düzenlememiz gerekiyor. Terminalde **"sudo gedit /etc/hosts"** yazdıktan sonra resimdeki gibi bilgisayarların ip numaralarını ve isimlerini yazıyoruz. Master ve slave bütün bilgisayarlarda aynı olamalıdır.

![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_etc_hosts.png)

ben burada sanal1 isimli cihazı master olarak, sanal2 ve sanal3 isimli cihazları slave olarak kullanacağım.

ssh ayarları ile devam ediyoruz.

	ssh-keygen -t rsa -P ""   # master ve slave bütün bilgisayarlarda bu komut çalıştırılacak.

SSH keylerini master bilgisayara kopyalıyoruz.

	cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys           #master yazacak
	ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@sanal1                #master yazacak
	ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@sanal2                #master yazacak
	ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@sanal3                #master yazacak
	.
	.
	.
	
Kaç tane bilgisayar kullanacaksanız onlarıda eklemelisiniz.

İşlemler bittikten sonra tüm bilgisayarlara ssh ile şifre sormadan bağlanabilmemiz gerekmektedir. ***"ssh sanal2"*** yazarak test edebiliriz. Şifre sormadan bu bilgisayarlara bağlanabiliryorsak devam edebiliriz.


## 4.adım

https://archive.apache.org/dist/hadoop/core/hadoop-2.7.2/hadoop-2.7.2.tar.gz adresindeki dosyayı indirelim. Ve arşivden çıkan klasörü **"/home/hadoop"** klasörüne taşıyalım.

![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_ls_hadoop.png)


Klasörü kopyaladıktan sonra terminalde **"gedit ~/.bashrc"** yazalım açılan .bashrc dosyasına aşağıdaki komutları ekleyelim.

	export HADOOP_PREFIX=/home/hadoop/hadoop-2.7.2
	export PATH=$PATH:$HADOOP_PREFIX/bin
	export PATH=$PATH:$HADOOP_PREFIX/sbin
	export HADOOP_MAPRED_HOME=${HADOOP_PREFIX}
	export HADOOP_COMMON_HOME=${HADOOP_PREFIX}
	export HADOOP_HDFS_HOME=${HADOOP_PREFIX}
	export YARN_HOME=${HADOOP_PREFIX}
	export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_PREFIX/lib/native
	export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib"
	
**.bashrc** dosyasını düzenleyerek kaydettikten sonra terminalde **"source .bashrc"** yazıyoruz.

Şimdi kontol edelim. Buraya kadar işlemleri doğru yaptıysanız **"hadoop version"** yazında aşağıdaki gibi çıktınız olacaktır.

![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_hadoop_version.png)

Şimdi bazı config dosyalarının içeriğini düzenleyeceğiz. 

/home/hadoop/hadoop-2.7.2/etc/hadoop/hadoop-env.sh ***#master  ve slave için aynı
Dosya içerisinde bir satır düzenlenecek.***

	export JAVA_HOME=/usr/lib/jvm/java-8-oracle/

/home/hadoop/hadoop-2.7.2/etc/hadoop/core-site.xml ***#master  ve slave için aynı**

	<configuration>
	<property>
	<name>fs.default.name</name>
	<value>hdfs://sanal1:9000</value>
	</property>
	</configuration>

/home/hadoop/hadoop-2.7.2/etc/hadoop/hdfs-site.xml ***#master için***

	<property>
	<name>dfs.replication</name>
	<value>2</value>
	</property>
	<property>
	<name>dfs.permissions</name>
	<value>false</value>
	</property>
	<property>
	<name>dfs.namenode.name.dir</name>
	<value>/home/hadoop/hadoop-2.7.2/namenode</value>
	</property>
	<property>
	<name>dfs.datanode.data.dir</name>
	<value>/home/hadoop/hadoop-2.7.2/datanode</value>
	</property>
	</configuration>
	
/home/hadoop/hadoop-2.7.2/etc/hadoop/hdfs-site.xml ***#slave için***	
	
	<configuration>
	<property>
	<name>dfs.replication</name>
	<value>2</value>
	</property>
	<property>
	<name>dfs.permissions</name>
	<value>false</value>
	</property>
	<property>
	<name>dfs.datanode.data.dir</name>
	<value>/home/hadoop/hadoop-2.7.2/datanode</value>
	</property>
	</configuration>
	
	
/home/hadoop/hadoop-2.7.2/etc/hadoop/mapred-site.xml ***#master  ve slave için aynı
bu dosya varsayılan olarak yok. "mapred-site.xml.template"  dosyasını kopyalayarak adını değiştiriyoruz.***

	<configuration>
	<property>
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
	</property>
	</configuration>

/home/hadoop/hadoop-2.7.2/etc/hadoop/yarn-site.xml ***#master  ve slave için aynı***

	<configuration>
	<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
	</property>
	<property>
	<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
	<value>org.apache.hadoop.mapred.ShuffleHandler</value>
	</property>
	<property>
	<name>yarn.resourcemanager.resource-tracker.address</name>
	<value>sanal1:8025</value>
	</property>
	<property>
	<name>yarn.resourcemanager.scheduler.address</name>
	<value>sanal1:8030</value>
	</property>
	<property>
	<name>yarn.resourcemanager.address</name>
	<value>sanal1:8040</value>
	</property>
	</configuration>
	
/home/hadoop/hadoop-2.7.2/etc/hadoop/slaves  ***#master için 
Bu dosyaya slave olacak bilgisayarları ekliyoruz.(Ben master bilgisayarı yazmadım isterseniz eklenebilir).***

	sanal2
	sanal3

Şimdi terminalde **"/home/hadoop/hadoop-2.7.2"** klasörüne geliyoruz.


Şimdi hadoop'u başlatalım.Ssırası ıle bu komutları yazıyoruz.

	./sbin/start-dfs.sh
	./sbin/start-yarn.sh
	
İlk olarak namenodu oluşturuyoruz. 
	
	hdfs namenode -format
	
Bu işlemden sonra **"/home/hadoop/hadoop-2.7.2"** içinde namenode adlı bir klasör olması gerekmekte. Bu işlem sonrası hadoop'u durdurup tekrar başlatalım.

	./sbin/stop-yarn.sh
	./sbin/stop-dfs.sh
	
	./sbin/start-dfs.sh
	./sbin/start-yarn.sh



Eğer herşey doğru ise:
	
* Master bilgisayarda "jps" yadıktan sonra terminalde çıktı olarak bunları görmemiz gerekmekte.


		ResourceManager
		Jps
		NameNode
		NodeManager
		
	(eğer slaves dosyasına master bigisayarınıda ekledıysek bunlara ek olarak "DataNode" da görmemiz gerekli.)
	
	![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_master_jps.png)
	
* Slave bilgisayarda "jps" yadıktan sonra terminalde çıktı olarak bunları görmemiz gerekmekte.
	
		NodeManager
		DataNode
		Jps
		
	![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_slave_jps.png)
	
	
Birde webtarayıcımızda bakalım. 

* "http://sanal1:8088" adresıne gidelim ve sol taraftan nodes kısmına tıklayalım. Resımdeki gibi slave nodelar listelenmelidir.

![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_sanal1_8088.png) 

* "http://sanal1:50070" adresıne gidelim ve üst taraftan DataNodes kısmına tıklayalım. Resımdeki gibi datanodelar listelenmelidir.

![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_sanal1_50070.png) 


Buraya aşamada resimlerdeki gibi bir sonuç aldıysanız başarıyla kurulum gerçekleştirilmiştir.

Şimdi hadoop'un örnek olarak veriği wordcount uygulamasını deneyelim.
önce verdiğim blogs klasörünü masaüstüne kopyalayalım.
![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_desktop.png) 

Hadoop çalışırken bu komutları yazalım.
	
	bin/hdfs dfs -mkdir /user
	bin/hdfs dfs -mkdir /user/hadoop
	bin/hdfs dfs -put /home/hadoop/Desktop/blogs input
	
![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_datanode_input.png) 

Şimdi jar dosyamızı çalıştıralım.

	bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount input output
	
![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_application.png) 

İşlem bitince sonuçları görebiliriz.

	bin/hadoop dfs -cat output/*
![](https://github.com/satilmisyusuf/Hadoop-Install-and-Example/blob/master/images/img_output.png) 

