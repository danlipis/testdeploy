# argentum-web
Conector HDFS
O conector HDFS permite exportar dados dos tópicos do Kafka para arquivos HDFS. Este dados podem ser entregues para outros sistemas como; Elasticsearch, Hadoop e qualquer tipo de banco de dados.
Pré-requisitos	
É necessário que Zookeeper, Kafka e SchemaRegistry, já tenham sido inicializados. As instruções para subir estes outros estão disponíveis em outras documentações no Confluence. É preciso também  utilizar a plataforma do Hadoop, local ou remotamente e este reconheça a URL em que o HDFS esta publicando. A o Hive precisa instar instalado para interação com Hadoop.
Hadoop é uma plataforma de software de código aberto para o armazenamento e processamento distribuído de grandes conjuntos de dados, utilizando clusters de computadores com hardware commodity
Hive permite que os usuários de negócios e analistas de dados para usar suas análises de negócios preferenciais, relatórios e ferramentas de visualização com Hadoop

Guia básico de instalação do HDSF
A proposta é usar o conector HDSF para exportar dados gerados dentro da console de Producer.
1.	Iniciar um Producer Avro definindo Schema:
$./bin/kafka-avro-console-producer --broker-list localhost:9092 --topic test_hdfs \
--property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"f1","type":"string"}]}'
*O comando kafka-avro-console-producer é um Producer de linha de comando para ler dados de entrada padrão da console e escrever em um tópico Kafka em uma avro formato. 

2.	Gerar registro dentro da console conforme Schema definido
{"f1": "value1"}
{"f1": "value2"}
{"f1": "value3"}
*Os três registros inseridos serão publicados no tópico Kafka test_hdfs no formato Avro.
3.	Criar um diretório para HDFS. 
$ hadoop fs -mkdir /topics/test_hdfs/partitions=0




4.	Iniciarlizar conexão do Kafka Connect com Conector HDFS:

$ ./bin/connect-standalone /etc/schema-registry/connect-avro-standalone.properties \
/etc/kafka-connect-hdfs/quickstart-hdfs.properties
*Durante o processo de inicialização será exibida algumas mensagens e, em seguida, ele exportará dados do Kafka para o HDFS. 
5.	Para validar se os dados foram copiados para dentro do diretório HDFS,  utilize o comando hadoop para listar o conteúdo do diretório:
$ hadoop fs -ls /topics/test_hdfs/partitions=0
*Deve ser mostrado o arquivo  com o nome /topics/t1/partition=0/test_hdfs+0+0000000000+0000000002.avro.  O nome do arquivo está codificado como topic+partition+start_offset+end_offset.format.
6.	Para extrair  o contéudo do arquivo pode ser utilizado avro-tools-1.7.7.jar:
$ hadoop jar avro-tools-1.7.7.jar tojson /topics/test_hdfs/partition=0/test_hdfs+0+0000000000+0000000002.avro

Configurações do arquivo HDFS
Esta seção mostra propriedades do arquivo de configurçaõ do HDFS utilizadas no guia basíco de instalação.
Contéudo do arquivo /etc/kafka-connect-hdfs/quickstart-hdfs.properties
name=hdfs-sink
connector.class=io.confluent.connect.hdfs.HdfsSinkConnector
tasks.max=1
topics=test_hdfs
hdfs.url=hdfs://localhost:9000
flush.size=3

•	name  é um nome criado pelo usuário para a instância do conector;
•	connector.class especifica a classe de implementação, basicamente o tipo de conector;
•	task.max especifica quantas instâncias do nosso conector de origem devem ser executadas em paralelo;
•	topic define o tópico para o qual o conector deve enviar a saída;
•	hdfs.url especifica o HDFS no qual esta escrevendo;
•	flush.size especifica o número de registros para escrever antes do commit do arquivo para HDFS;

*A classe de conector utilizado “io.confluent.connect.hdfs.HdfsSinkConnector” é do tipo sink connector.  Este entrega dados de tópicos no Kafka para outros sistemas, podendo ser índices, como o Elasticsearch, sistemas em lote como o Hadoop ou qualquer tipo de banco de dados.
