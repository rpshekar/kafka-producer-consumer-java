# Kafka Confluent .NET  producer and consumer example with encryptions 

Produce encrypt messages to and consume decrypt messages from a Kafka cluster using the .NET Producer and Consumer example.


## Build Nuget Packages
* https://www.nuget.org/packages/Confluent.Kafka/
* Confluent.Kafka 
    * https://docs.confluent.io/kafka-clients/dotnet/current/overview.html


	

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

## DLLs

* The following DLLs have been provided to cardinal by the confluent accelerator program and must be included in the project

	Confluent.Encryption.Azure.dll
	Confluent.Encryption.Common.dll
	Confluent.Encryption.Serializer.dll

## Conflunet Broker Server Connection and Consumer/Producer KMS Properties  in appsettings.json
	{
	 "Logging": {
    "LogLevel": {
      "Default": "Information",
      "System.Net.Http.HttpClient": "Warning",
      "Microsoft.AspNetCore": "Warning"
    }
	},

	"KafkaConsumerConfig": {
    "BootstrapServers": "localhost:9092", or <confluent-server:port>
    "SaslUserName": "XXXXXXXX",
    "SaslPassword": "XXXXXXCCCCCCC",
    "GroupId": "polaris-dev-appservice-consumer-group",
    "AutoOffsetReset": "AutoOffsetReset.Earliest",
    "SslCaLocation": "cacert.pem"
    },

	"KafkaEncryptionConfig": {
    "AzureVault": "test-kv-dev",
    "AzureClientId": "xxxxxx",
    "AzureClientSecret": "xxxxxx",
    "AzureTenantId": "xxxxxx",
    "AzureVaultKeyName": "test-Dev-Key"
	}
	}

## Producer example
 

* we use dependency injection to request producer instances and dispatch encrypted messages into a defined topic.

* Create HelloWorldProducer by extending the base KafkaProduccer class. Here we also define the type of the key for the message as a string
~~~
public class HelloWorldProducer : KafkaProducer<string, HelloWorldEvent>
	{
    public HelloWorldProducer(ProducerConfig config,
        IOptions<KafkaEncryptionConfig> kafkaEncryptionConfig,
        IOptions<FeaturesConfig> featuresConfig
    ) : base(config, kafkaEncryptionConfig, featuresConfig)
    {
      // implementation 
    }
	}
~~~

* Define producer instance using dependency injection
	
	services.AddScoped<IKafkaProducer<string, HelloWorldEvent>, HelloWorldProducer>();

* Get an instance of producer from DI and create an event
~~~
	public class HelloWorldController : ControllerBase
	{
    private readonly IKafkaProducer<string, HelloWorldEvent> _helloWorldProducer;
    public HelloWorldController(IKafkaProducer<string, HelloWorldEvent> helloWorldProducer)
    {
        _helloWorldProducer = helloWorldProducer;
    }
    [HttpPost]
    [Consumes(MediaTypeNames.Application.Json)]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<IActionResult> Hello(HelloRequest request)
    {
        // produce hello world event
        await _helloWorldProducer.ProduceAsync(
            topic: KafkaTopics.HelloWorldTopic,
            key: Guid.NewGuid().ToString(),
            value: new HelloWorldEvent
            {
                Name = request.name
            }
        );
        return Ok("Message Produced");
    }
}
~~~

## Producer Encryption Serializer Configuration
	
* KafkaSerializer.cs configures an encryption strategy leveraging Azure Key Vault.
~~~
	var azureProvider = AzureKmsCipherProviderBuilder.NewBuilder()
    .WithName("azureProvider")
    .WithVault(_kafkaEncryptionConfig.AzureVault)
    .WithClientId(_kafkaEncryptionConfig.AzureClientId)
    .WithClientSecret(_kafkaEncryptionConfig.AzureClientSecret)
    .WithTenantId(_kafkaEncryptionConfig.AzureTenantId)
    .Build();
	var masterCipherProvider = CacheCipherProviderBuilder.NewBuilder()
    .WithName("azure")
    .WithNestedProvider(azureProvider)
    .Build();
	var localCipherProvider = LocalCipherProviderBuilder.NewBuilder()
    .WithName("local")
    .Build();
	var generatorCipherProvider = GeneratorCipherProviderBuilder.NewBuilder()
    .WithName("generator")
    .WithNestedProvider(localCipherProvider)
    .Build();
	var dataCipherProvider = CacheCipherProviderBuilder.NewBuilder()
    .WithName("cache")
    .WithExpiry(10)
    .WithNestedProvider(generatorCipherProvider)
    .Build();
	SecuredSerializers.SecuredUtf8 serializer =
    SecuredSerializers.SecuredUtf8.WithConfiguration(
        ConfigurationsBuilder.NewBuilder()
            .WithConfiguration(new EncryptionConfiguration(
                new CipherConfiguration(new CipherProperties("GeneratedKey")),
                new CipherConfiguration(new CipherProperties(_kafkaEncryptionConfig.AzureVaultKeyName))
                    {ProviderName = "azure"},
                true,
                false
            ))
            .Build(),
        EncryptionBuilder.NewBuilder()
            .WithDefaultProvider(dataCipherProvider)
            .WithProvider(masterCipherProvider)
            .WithFallbackToDefaultProvider(true)
            .Build());
	return serializer;
~~~	

## Consumer example
* we use dependency injection to request consumer instances and register handlers to handle events from a defined topic.


* Create HelloWorldConsumer by extending the base KafkaConsumer class. Here we also define the type of the key for the message as a string
~~~

    public class HelloWorldConsumer : BaseConsumer
		{
		private readonly IKafkaConsumer<string, HelloWorldEvent> _consumer;
		public HelloWorldConsumer(IKafkaConsumer<string, HelloWorldEvent> kafkaConsumer)
		{
		_consumer = kafkaConsumer;
		}
		protected override async Task ExecuteAsync(CancellationToken stoppingToken)
		{
		try
		{
			await _consumer.Consume(KafkaTopics.HelloWorldTopic, stoppingToken);
		}
		catch (Exception ex)
			{
			// $"{(int) HttpStatusCode.InternalServerError} ConsumeFailedOnTopic - {KafkaTopics.HelloWorldTopic }, {ex}"
			}
		}
		public override void Dispose()
		{
			_consumer.Close();
			_consumer.Dispose();
			base.Dispose();
		}
	}
~~~
* Define consumer instance using dependency injection
~~~
	services.AddHostedService<HelloWorldConsumer>();
	services.AddScoped<IKafkaHandler<string, HelloWorldEvent>, HelloWorldHandler>();
~~~
* The message is handled in the HelloWorldHandler.cs as registered in the DI

## Consumer Decryption Serializer Configuration

*	In the demo application KafkaDeserializer.cs configures an decryption strategy leveraging Azure Key Vault. It also configures Cipher cashing which avoids round trips to Azure Key Vault.
~~~
	var azureProvider = AzureKmsCipherProviderBuilder.NewBuilder()
    .WithName("azureProvider")
    .WithVault(_kafkaEncryptionConfig.AzureVault)
    .WithClientId(_kafkaEncryptionConfig.AzureClientId)
    .WithClientSecret(_kafkaEncryptionConfig.AzureClientSecret)
    .WithTenantId(_kafkaEncryptionConfig.AzureTenantId)
    .Build();
	var masterCipherProvider = CacheCipherProviderBuilder.NewBuilder()
    .WithName("azure")
    .WithNestedProvider(azureProvider)
    .Build();
	var localCipherProvider = LocalCipherProviderBuilder.NewBuilder()
    .WithName("local")
    .Build();
	var dataCipherProvider = CacheCipherProviderBuilder.NewBuilder()
    .WithName("cache")
    .WithNestedProvider(localCipherProvider)
    .Build();
	SecuredDeserializers.SecuredUtf8 deserializer =
    SecuredDeserializers.SecuredUtf8.WithConfiguration(
        ConfigurationsBuilder.NewBuilder()
            .WithConfiguration(new EncryptionConfiguration(
                new CipherConfiguration(new CipherProperties("GeneratedKey")),
                new CipherConfiguration(new CipherProperties(_kafkaEncryptionConfig.AzureVaultKeyName))
                    {ProviderName = "azure"},
                true, 
                false
            ))
            .Build(),
        EncryptionBuilder.NewBuilder()
            .WithDefaultProvider(dataCipherProvider)
            .WithProvider(masterCipherProvider)
            .WithFallbackToDefaultProvider(true)
            .Build());
	return deserializer;
~~~	
	
## Appendix


* Producer: a process which publishes messages to a Kafka topic


* Consumer: a process which subscribes to a Kafka topic, reads it’s messages, and commits offsets upon successfully processing it’s messages


* BootstrapServers
Initial list of brokers as a CSV list of broker host or host:port.


* SaslUsername and SaslPassword
Username and password for use with the PLAIN and SASL-SCRAM-.. mechanism


* SaslMechanisim and SecurityProtocol
SASL mechanism and protocol to use for authentication


* GroupId
Client group id string. All clients sharing the same group id belong to the same group.


* AutoOffsetReset
Action to take when there is no initial offset in offset store or the desired offset is out of range. Possible options are Earliest (automatically reset the offset to the smallest offset), Latest (automatically reset the offset to the largest offset), and Error (trigger an error)
   
  
&copy; This is copyright Cardinal health 
 
