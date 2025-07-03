# Changelog

Todas as mudanças notáveis neste projeto serão documentadas neste arquivo.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/),
e este projeto adere ao [Versionamento Semântico](https://semver.org/lang/pt-BR/).

## [1.0.0] - 2025-07-02

### Adicionado
- Campo `kid` (Key ID) opcional na mensagem `SignetPayload` para identificação de chaves
- Padrão KeyResolver recomendado para validação em produção
- Seção "Implementando um KeyResolver Seguro" com recomendações de cache TTL
- Orientações sobre retrocompatibilidade para `kid` vazio
- Recomendações para tratamento de erro formal (`ErrUnknownKeyID`)

### Alterado
- Status do projeto de "Rascunho" para "v1.0 - Estável"
- Seção 11.1 reescrita para refletir a nova arquitetura baseada em KeyResolver
- Documentação atualizada para enfatizar o uso de função na API de validação

### Documentação
- Adicionada seção "Implementação de Referência" no README.md
- Link para o repositório signet-go como implementação oficial
- Melhorias na documentação do campo `kid` na seção 3.3

### Infraestrutura
- Adicionada licença MIT
- Criado guia de contribuição (CONTRIBUTING.md)
- Adicionado código de conduta (CODE_OF_CONDUCT.md)
- Configurados templates para Issues e Pull Requests
- Criado changelog para rastreamento de mudanças

---

## Notas de Lançamento v1.0.0

A especificação Signet v1.0 está agora **estável e finalizada**. Esta versão representa um marco importante no projeto, estabelecendo um padrão maduro e bem definido para segurança de aplicação em sistemas distribuídos.

### O que significa "Estável"

- **API Estável**: A estrutura do `SignetPayload` e `SignetToken` está congelada
- **Compatibilidade**: Implementações baseadas na v1.0 continuarão funcionando
- **Interoperabilidade**: Diferentes implementações podem trocar tokens com confiança
- **Documentação Completa**: Todas as seções estão finalizadas e validadas

### Implementação de Referência

A implementação oficial e validada da especificação v1.0 está disponível em:
- **[signet-go](https://github.com/lucas-de-lima/signet-go)** - Implementação de referência em Go

### Próximos Passos

- Implementações em outras linguagens são bem-vindas
- A comunidade pode começar a usar o Signet em produção
- Feedback e melhorias serão considerados para versões futuras
- A evolução da especificação seguirá o processo estabelecido no CONTRIBUTING.md

### Agradecimentos

Agradecemos a todos os contribuidores que participaram da definição desta especificação. O Signet v1.0 é o resultado de um esforço colaborativo para criar um padrão de segurança moderno, performático e interoperável.

---

**Nota**: Para mudanças futuras na especificação, consulte o [CONTRIBUTING.md](CONTRIBUTING.md) para entender o processo de evolução do padrão. 