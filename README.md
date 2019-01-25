# RabbitMQ demo

Este é um demo interativo mostrando como é feito a comunicação por filas no RabbitMQ. Criaremos duas classes Java: TestSender, que enviará uma mensagem **Hello World!** na fila,
e TestReceiver, que receberá essa mensagem.

![Exemplo de fila](queue_example.png)

## Mão na massa!

Aqui é Raiz! Aqui nós vamos criar tudo na unha com `javac` e `java`! Sem `maven` e essas frescuras! :D

```
Nota: Se estiver com esse repo clonado, vá direto para o passo 3.
```

 1 - Criar um novo diretório de trabalho

Crie seu novo diretório no seu lugar de preferencia:

```bash
mkdir rabbitmq-demo
cd rabbitmq-demo
```

 2 - Vamos também baixas todos os JARs que vamos precisar para utilizar o `amqp-client`. Digite:

```bash
wget http://central.maven.org/maven2/com/rabbitmq/amqp-client/5.5.3/amqp-client-5.5.3.jar
wget http://central.maven.org/maven2/org/slf4j/slf4j-simple/1.7.25/slf4j-simple-1.7.25.jar
wget http://central.maven.org/maven2/org/slf4j/slf4j-api/1.7.25/slf4j-api-1.7.25.jar
```

Ou então acesse o [mvn repository](https://mvnrepository.com) e encontre os JARs la. Escolha a versão mais recente.

```
Nota: Para facilitar o tutorial, esse repo já está com os JARs baixados. Se estiver com esse repo clonado, deixa pra lá e vá para o passo seguinte.
```

 3 - Vamos criar nosso broker local. Vamos utilizar o docker para isso:

```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

A porta 5672 é a porta do protocolo `AMQP`, enquanto que a porta `15672` é a porta para acessarmos o seu management via browser.

Com o broker criado na sua máquina, vamos começar a criar nossas classes Sender e Consumer!


### Criando o nosso Sender:

 1 - Crie um novo arquivo .java chamado `TestSender.java`

```bash
touch TestSender.java
```

 2 - Abra seu editor preferido e comece importando as classes do client RabbitMQ:

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
```

 3 - Ok, com essas classes importadas ja podemos começar a criar nossa classe principal e seu método `main`:

```java
 public class TestSender {
	public static void main(String[] args) throws Exception {
		// TODO: faremos muitas coisas legais aqui!
	}
}
```

 4 - Com a classe para o nosso sender criada, vamos começar a escrever dentro do método `main`. Primeiro, criaremos o nosso ConnectionFactory, essa classe é responsalve por criar uma conexão com broker RabbitMQ:

```java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
```

 5 - Para que possamos criar filas, producers, consumers e etc, precisamos criar uma nova conexão com o ConnectionFactory, e um Channel a partir dessa conexão criada. Esse channel é onde estão todos os comandos de comunicação com o broker:

```java
/*
Estamos utilizando o try-with-resources, que é um recurso do java7 em que 
você define um recurso (como um inputstream, uma conexão) que precise ser fechada
ao terminar de usá-la. Antigamente faríamos:

InputStream recurso = getInputStream();
try {
	readResource(recurso);
} finally {
	recurso.close();
}

Com o try-with-resources, você apenas define no início, e o recurs é automaticamente fechado,
sem a necessidade de um bloco finally boiler-plate:

try (InputStream recurso = getInputStream()) {
	readResource(recurso);
}

*/
try (Connection connection = factory.newConnection();

	// criando channel
	Channel channel = connection.createChannel()) {

	// TODO: Calma que já vamos escrever algo aqui!

}
```

 6 - Com o Channel criado agora é só criar a nossa fila para envio da mensagem com `queueDeclare`! Para isso, digite dentro do block `try`:

```java
// declarando fila com o nome "fila-teste"
String QUEUE = "fila-teste";
channel.queueDeclare(QUEUE, false, false, false, null);
```

Não se assuste com esses booleans `false`. Explicaremos esses parâmetros mais adiante!
Dúvida: E se a fila já existir? Tranquilo! Essa operação vai criar a fila, caso ela não exista, ou retorna OK, caso já exista!

 7 - Com a fila criada, é só enviar a mensagem com `basicPublish`!

```java
// enviando mensagem para fila
String message = "Hello World!";
channel.basicPublish("", QUEUE, null, message.getBytes());
System.out.println(" [x] Enviado '" + message + "'");
```

Note que só podemos enviar no formato `byte[]`, por isso obtemos os bytes da String criada e a passamos no método.

 8 - Beleza! Agora salve o arquivo e no shell, compile o arquivo java:

```bash
javac -cp amqp-client-5.5.3.jar TestSender.java
```

 9 - E, finalmente, vamos executar nossa classe:

```bash
java -cp .:amqp-client-5.5.3.jar:slf4j-api-1.7.25.jar:slf4j-simple-1.7.25.jar TestSender
```

Se sua saída foi algo parecido com ` [x] Enviado 'Hello World!'`, quer dizer que deu tudo certo! Parabéns!

E pronto! Você acaba de enviar sua primeira mensagem utilizando RabbitMQ! Vamos ver se enviou mesmo?

 1. Abra seu navegador e entre em `http://localhost:15672`
 2. Entre com seu usuário/senha (guest/guest, caso não tenha alterado)
 3. Acesse a aba `Queues`, e procure a fila `fila-teste`
 4. Verifique a quantidade de `Ready`
 5. Se estiver com quantidade "1", está correto!

Podemos verificar o que essa mensagem tem por dentro, para isso:

 1. Na aba `Queues`, clique na fila `fila-teste`,
 2. Clique em `Get messages`, logo abaixo
 3. Deixe com as opções como estão e clique no botão `Get Message(s)`
 4. Abaixo exibirá o conteúdo da mensagem (Hello World!)

Legal! Sua mensagem está na fila esperando ser consumida por algum consumer que não existe ainda. Então vamos criar!

### Criando nosso Consumer:

 1 - No diretório do projeto, crie uma nova classe java chamada `TestReceiver.java`

```bash
touch TestReceiver.java
```

 2 - Agora com seu editor de texto favorito, começaremos importando as classes necessárias:

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;
```

 3 - E a nossa class com o método `main`:

```java
public class TestReceiver {

	public static void main(String[] args) throws Exception {

		//TODO: já já!
	}
}
```

 4 - Mesma coisa feita no Sender, certo? Já está ficando acostumado com isso, hein? Vamos adiantar um pouco o passo então, precisamos criar a ConnectionFactory, a Connection e o Channel:

```java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");

try (Connection connection = factory.newConnection();
	 
	 // criando channel
	 Channel channel = connection.createChannel()) {
}
```

Lembra que o Channel é responsável por todos os comandos? Então, por isso que precisamos criar ele sempre!

 5 - Vamos declarar a fila (lembre-se que se já estiver criada ele não faz nada e retorna OK!):

```java
// declarando fila com o nome "fila-teste"
channel.queueDeclare(QUEUE, false, false, false, null);
```

 6 - Agora vem a coisa nova! Vamos criar um consumer, que é nada mais que um lambda que recebe dois parâmetros: `consumerTag` e `delivery`:

```java
// declarando consumidor da mensagem
DeliverCallback deliverCallback = (consumerTag, delivery) -> {
	String message = new String(delivery.getBody(), "UTF-8");
	System.out.println(" [x] '" + message + "'");
};
```

Nesse caso, apenas recebe a mensagem e imprime o seu `body`. Como `body` é um `byte[]`, passaremos esses bytes para montar uma String.

 7 - Agora é só consumir a fila com `basicConsume`! Iremos passar esse lambda para o método, que se houver algo na fila, irá passar para esse lambda. Vamos colocar também um `Thread.sleep` para evitar que se feche a conexão antes de consumir:

```java
// Consumindo da fila:
channel.basicConsume(QUEUE, true, deliverCallback, consumerTag -> { });
Thread.sleep(5000);
```

 8 - Salve o arquivo e no shell, vamos compilar a classe, exatamente como fazemos no Sender:

```bash
javac -cp amqp-client-5.5.3.jar TestReceiver.java
```

 9 - E, finalmente, vamos executar nosso consumer:

```bash
java -cp .:amqp-client-5.5.3.jar:slf4j-api-1.7.25.jar:slf4j-simple-1.7.25.jar TestReceiver
```

Se a saída foi: ` [x] 'Hello World!'`, parabèns! Você acabou de receber a mensagem enviada na fila!

Você pode verificar no management console que a fila ficou com 0 (zero) mensagens na fila!

E por aqui finalizamos nosso tutorial básico de envio/recebimento de mensagens na fila RabbitMQ!

## Que parâmetros são esses??

Se você viu o `queueDeclare`, viu que ele tem um monte de parâmetros booleanos. Vamos entender eles:

 * **boolean passive** = Se deixar `false`, o rabbit irá apenas checar se a fila existe com os mesmos parâmetros passados nesse método, retornando erro caso seja diferente. Se deixar `true`, o rabbitmq apenas verifica se existe.
 * **boolean durable** = Indica que essa fila é durável. Isso significa que, se o broker cair, ao subir novamente, sua fila estará la!
 * **boolean exclusive** = Exclusive quer dizer que essa fila será visível exclusivamente para esse Channel! Interessante para criar filas temporárias!
 * **boolean autoDelete** = Indica que essa fila deverá ser deletada após o fechamento da Channel! Também interessante para filas temporárias!

## Aprendi sobre filas, e agora?

Filas são o básico do básico no ambiente RabbitMQ. Existe muita coisa para estudar ainda, como **Exchanges**, **RPC**, **Publish/Subscribe**, boas práticas e mais! Enquanto esse tutorial
não contempla todos esses assuntos, recomendamos uma leitura no site do RabbitMQ:

[RabbitMQ tutorial](https://www.rabbitmq.com/getstarted.html)
