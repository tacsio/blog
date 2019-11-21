---
layout: article
title: Multi-Stage Build Docker + Spring Boot
date: 2019-11-20 21:33:00-0300
coverPhoto: https://tacsio.github.io/contents/images/2019/docker-boot.png
---

Então surge a ideia, utilizar o multi-stage build do Docker para embutir na criação da imagem a compilação e geração do 'executável' do Spring Boot, e as etapas para execução de fato da aplicação.

![cover][cover]

Em diversos tutoriais na internet, inclusive o pra própria [Pivotal][Pivotal], sempre vejo a necessidade de "existir" o .jar da aplicação já 'buildado'. Mas isso não é necessariamente interessante caso eu queira uma forma, mesmo em ambiente de desenvolvimento, realizar todo processo de compilação e 'startup' da minha aplicação.

Estudando mais a fundo Multi-Stage Builds, é possível unificar os famosos Dockerfile.build (para instruções estritamente de build da aplicação) com o Dockerfile utilizado para ambientes de produção.

## Chega de blá-blá-blá e Mãos à Obra

Primeiro vamos criar uma API bem simples:
{% gist a6f9adc8cc0d19d2f66e7ffed752e941 %}

E montar o Dockerfile para compilar e gerar o .Jar executável, para depois montar a imagem de execução da aplicação.
{% gist ab166223c320310715cb7cd3c6e06329 %}

Beleza, mas o que está acontecendo aqui?
1. Marcarmos a imagem do gradle:jdk8 com um 'label' de build
2. Montamos nosso ambiente de para o build da aplicação (copiando arquivos do projeto)
3. Como utilizei o gradle, basta executar o comando para gerar o build da aplicação (aqui também poderíamos parametrizar o docker file com ARG e utilizar os profiles do Spring)
4. Discartamos a imagem com os arquivos compilados, testes e afins (com o novo FROM openjdk:8-jdk-alpine)
5. Copiamos o que precisamos, o .JAR, do stage anterior e depois é só 'ladeira abaixo' com o padrão de sempre expor a porta e o ENTRYPOINT para executar o nosso jar

![gif][gif]


## Considerações Finais

Particularmente eu achei um pouco lenta a utilização do multi-stage no caso especifico do Spring Boot. Isso porque o build com Gradle do Spring em containers Docker é um pouco demorado (tem que baixar a internet todas as vezes e tal).

Em outras stacks é bem mais tranquilo. Build de aplicações React para produção + Nginxé o sucesso!


[[1] - Multi-Stage Build Docker][Docker-MultiStage]

[cover]: https://tacsio.github.io/contents/images/2019/docker-boot.png
[gif]: https://tacsio.github.io/contents/images/2019/docker-boot.gif
[Pivotal]: https://spring.io/guides/gs/spring-boot-docker/
[Docker-MultiStage]: https://docs.docker.com/develop/develop-images/multistage-build/
	