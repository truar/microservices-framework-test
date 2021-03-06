# Test of microservice framework

- Mac OsX High Sierra 10.13.6
- Java
- GraalVM

I try to get my hands on some new microservices framework we hear about since couple of months. 
My first ideas are 
- To test just Micronaut and Quarkus with Java 11, see if there "getting started" work
- Have fun with GraalVM and see if it is working, robust, and as fast as some might say !

Let's go.

## SDKMan installation
SDKMan is a great tool to manage different Software Development Kit you could need, like JVM, GraalVM, Ruby...

Here is the link to the [installation page](https://sdkman.io/install)
```
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk version
```

That's it. That's really, really fast.

Installing SDKMan is not mandatory, you could install GraalVM without it, but for the rest of this README,
I will use SDKMan to manage the GraalVM SDK installation

## Install GraalVM

GraalVM is a Polyglot Virtual Machine able to transform different programs written in different languages 
into a native program. The native program is supposed to very fast comparing to using others languages runtime
environment.

To install GraalVM to transform a JAR into a Shell:
```
sdk install java 19.3.0.r11-grl
java -version
```
> openjdk version "11.0.6" 2020-01-14<br/>
  OpenJDK Runtime Environment GraalVM CE 19.3.1 (build 11.0.6+9-jvmci-19.3-b07)<br/>
  OpenJDK 64-Bit Server VM GraalVM CE 19.3.1 (build 11.0.6+9-jvmci-19.3-b07, mixed mode, sharing)

The goal of this test is also to transform programs into native images. Therefor, you need to install the native-image tool:
```
gu install native-image
```

You're all set, ready to kick some ass !

## First test: Micronaut
Micronaut was the first framework I was introduced to, it was natural for me to try this one first.
I simply followed the ["getting started" guide](https://guides.micronaut.io/creating-your-first-micronaut-app/guide/index.html)

The apps was working on my IDE, also working when I was running the "FAT" jar, but not working when trying to run it as a native image.

Here are the commands:
```
cd micronaut
./gradlew assemble
java -jar build/libs/micronaut-0.1-all.jar
```
> 17:53:13.952 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 1651ms. Server Running: http://localhost:8080

```
curl http://localhost:8080/hello 
```
> Hello World

If you see `Hello World` then your program is working.

### Generating the native image
It is time to try to generate the native image, and see if it is working.
```
native-image -jar build/libs/micronaut-0.1-all.jar 
```
> Build on Server(pid: 10050, port: 59116)*
  [micronautguide:10050]    classlist:   4,309.74 ms<br/>
  [micronautguide:10050]        (cap):   2,006.14 ms<br/>
  [micronautguide:10050]        setup:   3,173.95 ms<br/>
  Warning: class initialization of class io.netty.handler.ssl.JettyAlpnSslEngine$ServerEngine failed with exception java.lang.NoClassDefFoundError: org/eclipse/jetty/alpn/ALPN$Provider. This class will be initialized at run time because option --allow-incomplete-classpath is used for image building. Use the option --initialize-at-run-time=io.netty.handler.ssl.JettyAlpnSslEngine$ServerEngine to explicitly request delayed initialization of this class.<br/>
  Warning: class initialization of class io.netty.handler.ssl.JettyAlpnSslEngine$ClientEngine failed with exception java.lang.NoClassDefFoundError: org/eclipse/jetty/alpn/ALPN$Provider. This class will be initialized at run time because option --allow-incomplete-classpath is used for image building. Use the option --initialize-at-run-time=io.netty.handler.ssl.JettyAlpnSslEngine$ClientEngine to explicitly request delayed initialization of this class.<br/>
  [micronautguide:10050]   (typeflow):  24,806.49 ms<br/>
  [micronautguide:10050]    (objects):  22,615.73 ms<br/>
  [micronautguide:10050]   (features):   3,958.32 ms<br/>
  [micronautguide:10050]     analysis:  54,801.15 ms<br/>
  [micronautguide:10050]     (clinit):   1,186.88 ms<br/>
  [micronautguide:10050]     universe:   3,777.48 ms<br/>
  [micronautguide:10050]      (parse):   3,540.40 ms<br/>
  [micronautguide:10050]     (inline):   4,543.24 ms<br/>
  [micronautguide:10050]    (compile):  30,930.13 ms<br/>
  [micronautguide:10050]      compile:  42,383.00 ms<br/>
  [micronautguide:10050]        image:   6,376.77 ms<br/>
  [micronautguide:10050]        write:   1,616.93 ms<br/>
  [micronautguide:10050]      [total]: 116,775.54 ms<br/>

With the warning, I am not very confident. I was right... When I start the new native image, 
I get an error
```
./micronautguide
```
> 17:58:24.520 [main] ERROR io.micronaut.runtime.Micronaut - Error starting Micronaut server: The offset of private final java.util.Set sun.nio.ch.SelectorImpl.selectedKeys is accessed without the field being first registered as unsafe accessed. Please register the field as unsafe accessed. You can do so with a reflection configuration that contains an entry for the field with the attribute "allowUnsafeAccess": true. Such a configuration file can be generated for you. Read CONFIGURE.md and REFLECTION.md for details.<br/>
  com.oracle.svm.core.jdk.UnsupportedFeatureError: The offset of private final java.util.Set sun.nio.ch.SelectorImpl.selectedKeys is accessed without the field being first registered as unsafe accessed. Please register the field as unsafe accessed. You can do so with a reflection configuration that contains an entry for the field with the attribute "allowUnsafeAccess": true. Such a configuration file can be generated for you. Read CONFIGURE.md and REFLECTION.md for details.<br/>
          at com.oracle.svm.core.util.VMError.unsupportedFeature(VMError.java:101)<br/>
          at jdk.internal.misc.Unsafe.objectFieldOffset(Unsafe.java:50)<br/>
          at sun.misc.Unsafe.objectFieldOffset(Unsafe.java:640)<br/>
          at io.netty.util.internal.PlatformDependent0.objectFieldOffset(PlatformDependent0.java:504)<br/>
          at io.netty.util.internal.PlatformDependent.objectFieldOffset(PlatformDependent.java:650)<br/>
          at io.netty.channel.nio.NioEventLoop$4.run(NioEventLoop.java:225)<br/>
          at java.security.AccessController.doPrivileged(AccessController.java:81)<br/>
          at io.netty.channel.nio.NioEventLoop.openSelector(NioEventLoop.java:215)<br/>
          at io.netty.channel.nio.NioEventLoop.<init>(NioEventLoop.java:147)<br/>
          at io.netty.channel.nio.NioEventLoopGroup.newChild(NioEventLoopGroup.java:138)<br/>
          at io.netty.channel.nio.NioEventLoopGroup.newChild(NioEventLoopGroup.java:37)<br/>
          at io.netty.util.concurrent.MultithreadEventExecutorGroup.<init>(MultithreadEventExecutorGroup.java:84)<br/>
          at io.netty.util.concurrent.MultithreadEventExecutorGroup.<init>(MultithreadEventExecutorGroup.java:58)<br/>
          at io.netty.util.concurrent.MultithreadEventExecutorGroup.<init>(MultithreadEventExecutorGroup.java:47)<br/>
          at io.netty.channel.MultithreadEventLoopGroup.<init>(MultithreadEventLoopGroup.java:59)<br/>
          at io.netty.channel.nio.NioEventLoopGroup.<init>(NioEventLoopGroup.java:78)<br/>
          at io.netty.channel.nio.NioEventLoopGroup.<init>(NioEventLoopGroup.java:73)<br/>
          at io.netty.channel.nio.NioEventLoopGroup.<init>(NioEventLoopGroup.java:60)<br/>
          at io.micronaut.http.server.netty.NioEventLoopGroupFactory.createEventLoopGroup(NioEventLoopGroupFactory.java:70)<br/>
          at io.micronaut.http.server.netty.NettyHttpServer.newEventLoopGroup(NettyHttpServer.java:498)<br/>
          at io.micronaut.http.server.netty.NettyHttpServer.createWorkerEventLoopGroup(NettyHttpServer.java:392)<br/>
          at io.micronaut.http.server.netty.NettyHttpServer.start(NettyHttpServer.java:243)<br/>
          at io.micronaut.http.server.netty.NettyHttpServer.start(NettyHttpServer.java:96)<br/>
          at io.micronaut.runtime.Micronaut.lambda$start$2(Micronaut.java:75)<br/>
          at java.util.Optional.ifPresent(Optional.java:183)<br/>
          at io.micronaut.runtime.Micronaut.start(Micronaut.java:73)<br/>
          at io.micronaut.runtime.Micronaut.run(Micronaut.java:307)<br/>
          at io.micronaut.runtime.Micronaut.run(Micronaut.java:293)<br/>
          at example.micronaut.Application.main(Application.java:8)<br/>

This doesn't feel right... I will have to come back and see if I can get something.


### And with Java 8 ?
Well, it is working with Java 8. Here are the steps to reproduce
```
# Go to micronaut dir
cd micronaut
# Install JDK 8 with GraalVM
sdk install java 19.3.0.r8-grl
# Install the tool native-image
gu install native-image
# Generate the FAT jar
./gradlew assemble
# Generate the native image
native-image --no-server -jar build/libs/micronaut-0.1-all.jar 
```
> [micronautguide:12218]    classlist:   4,765.04 ms<br/>
  [micronautguide:12218]        (cap):   1,809.87 ms<br/>
  [micronautguide:12218]        setup:   3,003.93 ms<br/>
  [micronautguide:12218]   (typeflow):  24,592.23 ms<br/>
  [micronautguide:12218]    (objects):  17,702.16 ms<br/>
  [micronautguide:12218]   (features):   2,629.99 ms<br/>
  [micronautguide:12218]     analysis:  48,321.01 ms<br/>
  [micronautguide:12218]     (clinit):   1,769.74 ms<br/>
  [micronautguide:12218]     universe:   3,174.54 ms<br/>
  [micronautguide:12218]      (parse):   4,808.69 ms<br/>
  [micronautguide:12218]     (inline):   5,795.80 ms<br/>
  [micronautguide:12218]    (compile):  37,931.60 ms<br/>
  [micronautguide:12218]      compile:  52,699.89 ms<br/>
  [micronautguide:12218]        image:   5,106.86 ms<br/>
  [micronautguide:12218]        write:   2,077.30 ms<br/>
  [micronautguide:12218]      [total]: 119,784.28 ms<br/>
  
With `19.3.0.r8-grl` I have no warnings... I am definitly more confident now. Let's run it
```
./micronautguide
```
> 18:31:56.962 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 34ms. Server Running: http://localhost:8080

Woooooww !! Did you see that ?! **36ms** to start what we thought was a heavy java web application. Impressive !

Is it working ? Let's find out
```
curl localhost:8080/hello
```
> Hello World

I am glad it works, but a bit disappointed it does not work with Java11. Still some way to go.

## Second test: Quarkus

Go on to this page to have the ["getting starded"](https://quarkus.io/guides/getting-started). 

Clearly, I appreciated a lot using Quarkus for this simple use case. Really easy to setup.

### Install the proper version of GraalVM
Quarkus seems to limit the usage of GraalVM to some specific version. If you want to build and execute the native image, you need to install `GraalVM 19.2.1`
```
sdk install java 19.2.1-grl
gu install native-image
java -version
```
> openjdk version "1.8.0_232"<br/>
  OpenJDK Runtime Environment (build 1.8.0_232-20191009173705.graal.jdk8u-src-tar-gz-b07)<br/>
  OpenJDK 64-Bit GraalVM CE 19.2.1 (build 25.232-b07-jvmci-19.2-b03, mixed mode)<br/>


For the rest of the commands, I assume you are in the `./quarkus` folder.

### Build a the JAR
To build the jar, do this
```
./mvnw install
java -jar target/getting-started-1.0-SNAPSHOT-runner.jar
```
> 2020-01-23 19:25:17,469 INFO  [io.quarkus] (main) getting-started 1.0-SNAPSHOT (running on Quarkus 1.1.1.Final) started in 0.610s. Listening on: http://0.0.0.0:8080<br/>
  2020-01-23 19:25:17,476 INFO  [io.quarkus] (main) Profile prod activated.<br/> 
  2020-01-23 19:25:17,476 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]<br/>

Let's see if this work:
```
curl localhost:8080/hello
```
> hello

That's great.

### Native image
In this short quick-start, what is great is that many things are already included into maven. To build the native image, just do:
```
./mvnw package -Pnative
```
> [INFO] [io.quarkus.deployment.QuarkusAugmentor] Quarkus augmentation completed in 50932ms<br/>
  [INFO] ------------------------------------------------------------------------<br/>
  [INFO] BUILD SUCCESS<br/>
  [INFO] ------------------------------------------------------------------------<br/>
  [INFO] Total time: 01:47 min<br/>
  [INFO] Finished at: 2020-01-23T19:09:25+01:00<br/>
  [INFO] ------------------------------------------------------------------------<br/>

Let's try it out.
```
./target/getting-started-1.0-SNAPSHOT-runner
```
> 2020-01-23 19:32:02,660 INFO  [io.quarkus] (main) getting-started 1.0-SNAPSHOT (running on Quarkus 1.1.1.Final) started in 0.013s. Listening on: http://0.0.0.0:8080<br/>
  2020-01-23 19:32:02,660 INFO  [io.quarkus] (main) Profile prod activated.<br/> 
  2020-01-23 19:32:02,660 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]<br/>

Even more WOOOOOOW !! **13ms** to start... Really impressive. Those native images get my attentions now !

Does it work ? of course !
``` 
curl localhost:8080
```
> hello

## Time for a debrief

I compared those 2 in a table

|                           | Micronaut                                                              | Quarkus                                           |
|---------------------------|------------------------------------------------------------------------|---------------------------------------------------|
| Maximum GraalVM supported | 19.3.0.r8-grl                                                          | 19.2.1-grl                                        |
| Java 11                   | Seems possible, but still not working                                  | Not even mentioned                                |
| Java 8                    | Working                                                                | Working                                           |
| FAT Jar suffix            | all                                                                    | runner                                            |
| Dependencies manager      | gradle                                                                 | maven                                             |
| Usage                     | Not so easy. Need more manual actions to build and have a native image | Very easy ! All works with and from maven         |
| Starting JAR              | Very fast: ~1.5s                                                       | Very fast: ~1.3s                                  |
| Starting native image     | Extremly fast : ~34ms                                                  | Extremly fast : ~15ms                             |
| Tests                     | Tests against controller                                               | Test against controller both jar and native image |
| Skeleton creation         | CLI                                                                    | Website

After discovering those 2 frameworks through a quick getting started, I must say I have a preference
for Quarkus. It seems faster, smaller, works well with Maven, and you have a 
[website to create an app skeleton](https://code.quarkus.io)

What I think is the more impressive for me is that how shirt is the uptime when building a native image.
You need more time at build, but you gain so much more for the runtime...

|                                    | JAR  | Native |
|------------------------------------|------|--------|
| Global build time                  | 7.7s | 107s   |
| Uptime                             | 1.3s | 0.012s |
| # of starts in 60s                 | 46   | 5000   |
| # of starts build included in 120s | 86   | 1083   |

Have fun ! Any comments are appreciated !