---
layout: article
title: Multi-Stage Build Docker + Spring Boot
date: 2019-11-21 21:33:00-0300
coverPhoto: https://tacsio.github.io/contents/images/2019/docker-boot.png
---

Então surge a ideia, utilizar o Multi-Stage Build do Docker para, na criação da imagem, incluir: compilação, geração do 'executável' Spring Boot e execução da aplicação.

![cover][cover]

Em diversos tutoriais na internet, inclusive o da própria [Pivotal][Pivotal], sempre vejo a necessidade de "existir" o .jar da aplicação já 'buildado'. Mas isso não é interessante caso eu queira uma forma, mesmo em ambiente de desenvolvimento, realizar todo processo de compilação e 'startup' da minha aplicação.

Estudando mais a fundo [Multi-Stage Builds][Docker-MultiStage], é possível unificar os famosos Dockerfile.build (para instruções estritamente de build da aplicação) com o Dockerfile utilizado para ambientes de produção.

## Chega de blá-blá-blá e Mãos à Obra

Primeiro vamos criar uma API bem simples:
```Java
package com.example.demo;

import lombok.Builder;
import lombok.Data;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}

@RestController
class DemoController{
	@GetMapping(value = "/{name}", produces = MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity hello(@PathVariable("name") String name) {
		String msg = String.format("May the Force by with you, %s", StringUtils.capitalize(name));
		return ResponseEntity.ok(JediResponse.builder().msg(msg).build());
	}
}

@Data
@Builder
class JediResponse {
	private String msg;
}
```

E montar o Dockerfile para compilar e gerar o .Jar executável, para depois montar a imagem de execução da aplicação.

```Dockerfile
# build environment
FROM gradle:jdk8 as build
WORKDIR /compile
COPY . /compile
RUN gradle build --no-daemon

#production environment
FROM openjdk:8-jdk-alpine
WORKDIR /app
VOLUME /tmp
COPY --from=build /compile/build/libs/demo-0.0.1-SNAPSHOT.jar api.jar
EXPOSE 8080
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app/api.jar"]
```

## Beleza, mas o que está acontecendo aqui?

1. Marcarmos a imagem do gradle:jdk8 com um 'label' de build
2. Montamos nosso ambiente de para o build da aplicação (copiando arquivos do projeto)
3. Como utilizei o gradle, basta executar o comando para gerar o build da aplicação (aqui também poderíamos parametrizar o docker file com ARG e utilizar os profiles do Spring)
4. Discartamos a imagem com os arquivos compilados, testes e afins (com o novo FROM openjdk:8-jdk-alpine)
5. Copiamos o que precisamos, o .JAR, do stage anterior e depois é só 'ladeira abaixo' com o padrão de sempre expor a porta e o ENTRYPOINT para executar o nosso jar

![gif][gif]


## Considerações Finais

Particularmente eu achei um pouco lenta a geração da imagem com Multi-Stage no caso especifico do Spring Boot. Isso porque o build com Gradle de aplicações Spring em containers Docker é um pouco demorado (tem que baixar a internet todas as vezes e tal).

Em outras stacks é bem mais tranquilo. Build de aplicações React para produção + Nginx é o sucesso!


[cover]: https://tacsio.github.io/contents/images/2019/docker-boot.png
[gif]: https://tacsio.github.io/contents/images/2019/docker-boot.gif
[Pivotal]: https://spring.io/guides/gs/spring-boot-docker/
[Docker-MultiStage]: https://docs.docker.com/develop/develop-images/multistage-build/
	