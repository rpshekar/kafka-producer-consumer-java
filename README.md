# kafka-producer-consumer-java
JAVA Kafka Confluent Producer /Consumer app
# kafka-producer-consumer-java
## Overview

Produce encrypt messages to and consume decrypt messages from a Kafka cluster using the Java Producer and Consumer example.

## Certifications to install Java key store in local
 Find certifications in java/cert directory
 Find run the keytool commands in ssl-commands.txt (Run from JRE or JDK Home/bin/)

## Set Class path library folder 
 Find jar files location in java/lib  
You can find the documentation and instructions for running this Java example at [https://docs.confluent.io/platform/current/tutorials/examples/clients/docs/java.html](https://docs.confluent.io/platform/current/tutorials/examples/clients/docs/java.html?utm_source=github&utm_medium=demo&utm_campaign=ch.examples_type.community_content.clients-ccloud)

## Commands 
* keytool -importcert -file C:\devtools\crt\encrypt-demo\prisma1_CAH_edge.cer -alias prisma2_edge -keystore cacerts -storepass changeit -noprompt
* keytool -importcert -file C:\devtools\crt\encrypt-demo\prisma2_edge.cer -alias prisma2_edge -keystore cacerts -storepass changeit -noprompt
* keytool -importcert -file C:\devtools\crt\encrypt-demo\lets-encrypt-r3-cross-signed.pem -alias lets-encrypt-r3-cross-signed  cacerts -storepass    changeit -noprompt

## Build Java using Maven 

	<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<parent>
		<groupId>io.confluent</groupId>
		<artifactId>rest-utils-parent</artifactId>
		<version>7.1.0</version>
	</parent>

	<artifactId>clients-example</artifactId>
	<packaging>jar</packaging>
	<version>7.1.0</version>

	<organization>
		<name>Confluent, Inc.</name>
		<url>http://confluent.io</url>
	</organization>
	<url>http://confluent.io</url>
	<description>
        Kafka Java clients examples for CCloud.
    </description>

	<properties>
		<java.version>8</java.version>
		<slf4j-api.version>1.7.6</slf4j-api.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<gson.version>2.2.4</gson.version>
		<schemaRegistryUrl>http://localhost:8081</schemaRegistryUrl>
		<schemaRegistryBasicAuthUserInfo></schemaRegistryBasicAuthUserInfo>
		<schemaLocal>src/main/resources/avro/io/confluent/examples/clients/cloud/DataRecordAvro2a.avsc</schemaLocal>
		<io.confluent.schema-registry.version>${confluent.version.range}</io.confluent.schema-registry.version>
	</properties>

	<licenses>
		<license>
			<name>Apache License 2.0</name>
			<url>http://www.apache.org/licenses/LICENSE-2.0.html</url>
			<distribution>repo</distribution>
		</license>
	</licenses>

	<repositories>
		<repository>
			<id>confluent</id>
			<name>Confluent</name>
			<url>https://packages.confluent.io/maven/</url>
		</repository>
	</repositories>

	<pluginRepositories>
		<pluginRepository>
			<id>confluent</id>
			<url>https://packages.confluent.io/maven/</url>
		</pluginRepository>
	</pluginRepositories>


	<dependencies>
		<dependency>
			<groupId>net.sf.opencsv</groupId>
			<artifactId>opencsv</artifactId>
			<version>2.3</version>
		</dependency>
		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>kafka-clients</artifactId>
			<version>${kafka.version}</version>
		</dependency>
		<dependency>
			<groupId>io.confluent</groupId>
			<artifactId>kafka-avro-serializer</artifactId>
			<version>${io.confluent.schema-registry.version}</version>
		</dependency>
		<dependency>
			<groupId>io.confluent</groupId>
			<artifactId>kafka-streams-avro-serde</artifactId>
			<version>${io.confluent.schema-registry.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>kafka-streams</artifactId>
			<version>${kafka.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>connect-runtime</artifactId>
			<version>${kafka.version}</version>
		</dependency>
		<dependency>
			<groupId>io.confluent</groupId>
			<artifactId>kafka-json-serializer</artifactId>
			<version>${io.confluent.schema-registry.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j-api.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j-api.version}</version>
		</dependency>
		<!-- Use a repackaged version of log4j with security patches. Default log4j v1.2 is a transitive dependency of slf4j-log4j12, but it is excluded in common/pom.xml -->
		<dependency>
			<groupId>io.confluent</groupId>
			<artifactId>confluent-log4j</artifactId>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>${jackson.version}</version>
		</dependency>
		<dependency>
			<groupId>com.google.code.gson</groupId>
			<artifactId>gson</artifactId>
			<version>${gson.version}</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.1</version>
				<inherited>true</inherited>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-assembly-plugin</artifactId>
				<configuration>
					<descriptorRefs>
						<descriptorRef>jar-with-dependencies</descriptorRef>
					</descriptorRefs>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>exec-maven-plugin</artifactId>
				<version>1.2.1</version>
				<executions>
					<execution>
						<goals>
							<goal>java</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.jasig.maven</groupId>
				<artifactId>maven-notice-plugin</artifactId>
				<version>1.0.6.1</version>
				<configuration>
					<licenseMapping>
						<param>../../license-mappings.xml</param>
					</licenseMapping>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.avro</groupId>
				<artifactId>avro-maven-plugin</artifactId>
				<version>${avro.version}</version>
				<executions>
					<execution>
						<phase>generate-sources</phase>
						<goals>
							<goal>schema</goal>
						</goals>
						<configuration>
							<sourceDirectory>${project.basedir}/src/main/resources/avro/io/confluent/examples/clients/cloud/</sourceDirectory>
							<includes>
								<include>DataRecordAvro.avsc</include>
							</includes>
							<outputDirectory>${project.build.directory}/generated-sources</outputDirectory>
						</configuration>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>io.confluent</groupId>
				<artifactId>kafka-schema-registry-maven-plugin</artifactId>
				<version>${io.confluent.schema-registry.version}</version>
				<configuration>
					<schemaRegistryUrls>
						<param>${schemaRegistryUrl}</param>
					</schemaRegistryUrls>
					<userInfoConfig>${schemaRegistryBasicAuthUserInfo}</userInfoConfig>
					<subjects>
						<test2-value>${schemaLocal}</test2-value>
					</subjects>
					<schemaTypes>
						<test2-value>AVRO</test2-value>
					</schemaTypes>
				</configuration>
				<goals>
					<goal>test-compatibility</goal>
				</goals>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-checkstyle-plugin</artifactId>
				<executions>
					<!--
                     This declaration merges with the one in the parent, rather
                     than overriding it, so we need to disable the "validate" phase
                     execution that the parent declares and declare our own
                     during "test-compile".

                     One reason for this is that avro codegen runs during compile,
                     and while it's not strictly a precondition, it's
                     confusing to address style violations while the IDE is telling you
                     that some generated class doesn't exist. Test-compile is the first phase
                     that's guaranteed to run after compile and before any unit or integration
                     tests.

                     Also, we want to disable the parent's configuration because it declares stuff
                     we don't care about, like suppressions. (Honestly, it shouldn't)
                     -->
					<execution>
						<id>validate</id>
						<phase>none</phase>
						<configuration>
							<skip>true</skip>
						</configuration>
					</execution>
					<execution>
						<id>test-compile</id>
						<phase>test-compile</phase>
						<configuration>
							<encoding>UTF-8</encoding>
							<consoleOutput>true</consoleOutput>
							<failsOnError>true</failsOnError>
							<failOnViolation>true</failOnViolation>
							<includeResources>false</includeResources>
							<includeTestResources>false</includeTestResources>
							<includeTestSourceDirectory>true</includeTestSourceDirectory>
							<excludes>io/confluent/examples/clients/cloud/**</excludes>
							<configLocation>checkstyle.xml</configLocation>
						</configuration>
						<goals>
							<goal>check</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
	

## Confluent Docker Setup 

* Docker link 

	curl --silent --output docker-compose.yml https://raw.githubusercontent.com/confluentinc/cp-all-in-one/7.2.0-post/cp-all-in-one/docker-compose.yml
`Docker compose ` <https://raw.githubusercontent.com/confluentinc/cp-all-in-one/7.2.0-post/cp-all-in-one/docker-compose.yml>
## Confluent Clould 
* bootstrap.servers=<Broker Url>
* ssl.endpoint.identification.algorithm=https
* security.protocol=SASL_SSL
* sasl.mechanism=PLAIN
* sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="{{API KEY}}" password="{{API SECRET}}";

## Local Docker  
 * bootstrap.servers=localhost:9092
 
## Consumer/Producer KMS  Properties
##### Local Encrypt with Symntrickkey
* bootstrap.servers=localhost:9092
* value.deserializer.key = RSA
* encryption.provider.name=local
* local.provider.class = io.confluent.encryption.common.crypto.cipher.impl.LocalCipherProvider
* local.provider.keys = RSA
* local.provider.RSA.key.type=SymmetricKey
* local.provider.RSA.key=hVLsA5I2/WOI37TDEgq+Fe9Iaf+BeGBzFHP1ywzJKWU= 

#####Azure
* value.serializer.key=Polaris-Dev-Key  
* encryption.provider.name=azure_provider 
* azure_provider.provider.class=AzureCipherProvider 
* azure_provider.provider.vault.name=polaris-kv-dev 
* azure_provider.provider.vault.client.id=<cliend id> 
* azure_provider.provider.vault.client.secret=<secret> 
* azure_provider.provider.vault.tenant.id=<tenent id> 
* azure_provider.provider.vault.custom.url=<url>

##### Azure envelope
* value.serializer.key = GeneratedKey
* value.serializer.wrapping.key = Polaris-Dev-Key
* value.serializer.wrapping.key.provider.name = azure
* azure.provider.class=io.confluent.encryption.common.crypto.cipher.impl.CachedCipherProvider
* azure.provider.expiry=3600
* azure.provider.name=azure_provider

## Producer example
 

	final Properties props = loadConfig(args[0]); // Load properties from  Consumer/Producer KMS  Properties
	props.put(ProducerConfig.ACKS_CONFIG, "all");
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "io.confluent.encryption.serializers.common.SecuredStringSerializer");
	 // Set Schema Registry 
	   // props.put("schema.registry.url", "https://<cloud>.gcp.confluent.cloud");
    //props.put("basic.auth.credentials.source", "USER_INFO");
      // props.put("basic.auth.user.info", "<userid>/<password>");
    while(true) { 
	   producer.send(new ProducerRecord<String, String>(topic, key, str), new Callback() {
          @Override
          public void onCompletion(RecordMetadata m, Exception e) {
            if (e != null) {
              e.printStackTrace();
            } else {
              System.out.printf("Produced record to topic %s partition [%d] @ offset %d%n", m.topic(), m.partition(), m.offset());
            }
          }
      });
    }
## Consumer example
  final Properties props = loadConfig(args[0]);

    // Add additional properties.
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
     // props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "io.confluent.encryption.serializers.common.SecuredStringDeserializer");
     // props.put(KafkaJsonDeserializerConfig.JSON_VALUE_TYPE, Example.class);
     props.put(ConsumerConfig.GROUP_ID_CONFIG, "demo-consumer-10");
     props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    // Schema configuration
      // props.put("schema.registry.url", "https://<cloud>.gcp.confluent.cloud");
      //props.put("basic.auth.credentials.source", "USER_INFO");
     // props.put("basic.auth.user.info", "<userid>/<password>");
    final Consumer<String, String> consumer = new KafkaConsumer<String, String>(props);
    consumer.subscribe(Arrays.asList(topic));
    try {
      while (true) {
        ConsumerRecords<String,  String> records = consumer.poll(100);
        for (ConsumerRecord<String,  String> record : records) {
        	Object key = record.key();
        	String obj= record.value();
         System.out.printf("Consumed record with key %s and city %s, %n", key, obj);
        }
      }
    } finally {
      consumer.close();
    }
  
&copy; This is copyright Cardinal health 
 
