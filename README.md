# Signet: Uma Especificação para Segurança de Aplicação

Signet é uma especificação aberta para a criação, assinatura e validação de cargas de segurança (payloads) de aplicação. Ela foi projetada para fornecer um padrão de identidade seguro, performático e interoperável, otimizado para ecossistemas de alta performance como gRPC, substituindo a necessidade de padrões legados baseados em texto.

## O Problema: A Segurança Adaptada

Por muito tempo, a indústria adaptou o JWT, um padrão da era REST/JSON, para ecossistemas binários. Essa adaptação nos trouxe um custo inerente:

- **Ineficiência:** A sobrecarga do parsing de JSON e da codificação Base64 em ambientes que anseiam por performance.
- **Insegurança por Padrão:** A flexibilidade de algoritmos fracos (`alg:none`) e a falta de um ciclo de vida de revogação claro deixam a maior parte da segurança como um exercício para o implementador.
- **Falta de Clareza Operacional:** A dificuldade de depurar e a falta de uma separação clara entre a identidade do transporte (mTLS) e a da aplicação.

## A Solução: Signet, Confiança Nativa

Signet não é um "JWT melhor". É uma resposta fundamentalmente diferente, projetada nativamente para o mundo em que opera. Ele não é apenas um token; é uma especificação que define um padrão de confiança.

## Os Pilares do Signet

- **Segurança por Design:** Algoritmos fortes são o padrão. A estrutura é definida por um contrato Protobuf, e a revogação é uma capacidade arquitetada, não uma reflexão tardia.
- **Performance Inerente:** Ao abraçar a codificação binária, o Signet elimina a sobrecarga desnecessária, operando na velocidade dos sistemas que protege.
- **Clareza Operacional:** Com ferramentas dedicadas para depuração e uma clara separação de responsabilidades, o Signet foi projetado para ser compreendido e operado com confiança.
- **Ecossistema Aberto:** Como uma especificação aberta, o Signet foi feito para ser implementado em múltiplas linguagens, garantindo a interoperabilidade que os microsserviços modernos exigem.

## Navegando neste Repositório

Este repositório é a fonte canônica da verdade para a especificação do Signet.

- **Para começar, leia o Manifesto:** Entenda a filosofia e a visão por trás do projeto. _(Nota: Link para o manifesto será adicionado aqui)_
- **Para detalhes técnicos, consulte a Especificação v1.0:** Este é o documento principal que detalha a estrutura, as regras e as considerações de segurança.
- [Leia a Especificação Signet v1.0](./SPECIFICATION-v1.0.md)

## Status do Projeto

**Status:** v1.0 - Estável

---

## Implementação de Referência

A implementação oficial e validada da especificação v1.0 está disponível em:

- [signet-go (GitHub)](https://github.com/signet/signet-go)

O repositório signet-go é mantido pela equipe principal do projeto e serve como referência para conformidade e interoperabilidade.

## Contribuição

Detalhes sobre como contribuir para a especificação serão adicionados em breve.