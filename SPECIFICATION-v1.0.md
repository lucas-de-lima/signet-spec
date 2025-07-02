# Especificação Signet v1.0

**Status:** Rascunho Inicial  
**Data:** 30 de junho de 2025

---

## 1. Resumo

Este documento define a especificação v1.0 do Signet, um padrão para cargas de segurança (payloads) de aplicação. O Signet foi projetado para fornecer um mecanismo de identidade de aplicação seguro, performático e interoperável, otimizado para ecossistemas de alta performance como gRPC.

---

## 2. Introdução e Terminologia

As palavras-chave **"DEVE" (MUST)**, **"NÃO DEVE" (MUST NOT)**, **"REQUER" (REQUIRED)**, **"DEVERIA" (SHOULD)**, **"NÃO DEVERIA" (SHOULD NOT)**, **"RECOMENDADO" (RECOMMENDED)**, **"PODE" (MAY)** e **"OPCIONAL" (OPTIONAL)** neste documento devem ser interpretadas como descrito na [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

- **Signet Payload:** A carga de dados contendo os claims de identidade e metadados.
- **Signet Token:** A estrutura final que encapsula o Signet Payload serializado e sua assinatura digital.
- **Emissor (Issuer):** A entidade que cria e assina o Signet Token.
- **Validador (Validator):** A entidade que verifica a integridade e validade do Signet Token.

---

## 3. Estrutura do Payload (`SignetPayload`)

O `SignetPayload` é a mensagem Protocol Buffers que contém o conjunto de claims (reivindicações) que formam a identidade da aplicação.

### 3.1. Claims Mandatórios

- **exp (Expiration Time):** Um timestamp Unix (número de segundos desde 1970-01-01T00:00:00Z UTC) que define o tempo de expiração do token. O token **NÃO DEVE** ser aceito para processamento após este tempo. A validação deste campo é obrigatória.
- **iat (Issued At):** Um timestamp Unix que define o tempo em que o token foi emitido. A validação deste campo é obrigatória e pode ser usada para determinar a idade do token.

### 3.2. Claims Recomendados

- **sub (Subject):** Uma string que representa o principal que é o sujeito do token (ex: ID do usuário). Sua interpretação é específica para o domínio da aplicação.
- **aud (Audience):** Uma string que identifica o(s) destinatário(s) do token. O Validador **DEVE** verificar se sua própria identidade corresponde à audiência declarada.

### 3.3. Claims Condicionais e Opcionais

- **sid (Session ID):** Um array de bytes contendo um identificador único para o token. Este campo é **REQUERIDO** para o perfil STATEFUL e **DEVE** estar vazio para o perfil STATELESS. O formato recomendado é um ULID ou UUIDv7 de 16 bytes.
- **custom_claims:** Um mapa de string para string para claims customizados, permitindo a extensão do payload com contexto específico da aplicação.
- **roles:** Uma lista de string para representar papéis ou escopos associados ao sujeito.
- **kid (Key ID):** Uma string opcional que identifica a chave usada para assinar o token. O campo `kid` permite que o validador selecione a chave pública correta para verificação, especialmente em cenários de rotação ou múltiplas chaves. Se ausente ou vazio, o validador DEVE utilizar a chave padrão configurada para o emissor.

---

## 4. Estrutura Final do Token (`SignetToken`)

O `SignetToken` é a estrutura final que é serializada e transportada. Ela garante a integridade do payload através de uma assinatura digital, separando claramente o conteúdo da prova criptográfica.

- **payload (bytes):** Contém a mensagem `SignetPayload` serializada para um array de bytes.
- **signature (bytes):** Contém a assinatura digital gerada a partir dos bytes do campo payload.

---

## 5. Arquivo de Definição Protobuf

A definição canónica e formal das estruturas de dados acima reside no arquivo `proto/v1/spec.proto`. Este documento fornece a prosa explicativa, mas o arquivo `.proto` é a fonte da verdade para a geração de código e a conformidade estrutural.

---

## 6. Visão Geral do Processo Criptográfico

O ciclo de vida de um `SignetToken` baseia-se num processo de assinatura e verificação simples e robusto. Um Emissor cria um `SignetPayload`, serializa-o e assina esses bytes para gerar uma `signature`. Ambos são encapsulados num `SignetToken`. Um Validador, ao receber o token, primeiro verifica a `signature` contra o payload serializado. Apenas se a assinatura for válida é que o Validador procede para deserializar e validar os claims contidos no payload. Este processo garante que nenhuma lógica de aplicação seja executada com base em um token cuja integridade não tenha sido criptograficamente comprovada. Os passos detalhados deste processo são descritos na seção seguinte.

---

# 7. Processo de Assinatura e Verificação

O ciclo de vida de um **SignetToken** é dividido em duas operações criptográficas principais: assinatura e verificação. A adesão estrita a este processo é mandatória para a conformidade com a especificação.

## 7.1. Criação e Assinatura

O processo de criação de um **SignetToken** por um Emissor **DEVE** seguir os seguintes passos:

1. **Construção do Payload:**
   - O Emissor **DEVE** construir uma instância da mensagem `SignetPayload` (`signet.v1.SignetPayload`), preenchendo todos os campos mandatórios (`exp`, `iat`) e os demais campos conforme a necessidade da aplicação e o perfil operacional escolhido.
2. **Serialização do Payload:**
   - A instância `SignetPayload` **DEVE** ser serializada para um array de bytes canônico, conforme definido pela especificação do Protocol Buffers. Este array de bytes será referido como `payload_bytes`.
3. **Assinatura Criptográfica:**
   - O Emissor **DEVE** usar o algoritmo **Ed25519** e sua chave privada para assinar os `payload_bytes`. O resultado desta operação é o `signature_bytes`.
4. **Construção do Token Final:**
   - O Emissor **DEVE** construir uma instância da mensagem `SignetToken` (`signet.v1.SignetToken`), inserindo os `payload_bytes` no campo `payload` e os `signature_bytes` no campo `signature`.
5. **Serialização Final:**
   - O `SignetToken` resultante é então serializado para um array de bytes final, pronto para ser transportado.

## 7.2. Verificação e Validação

O processo de verificação de um **SignetToken** por um Validador **DEVE** seguir os seguintes passos, e a falha em qualquer um deles **DEVE** resultar na rejeição do token:

1. **Deserialização do Token:**
   - O Validador recebe o array de bytes e **DEVE** deserializá-lo para uma instância da mensagem `SignetToken`. Se a deserialização falhar, o token é inválido.
2. **Verificação da Assinatura:**
   - O Validador **DEVE** usar a chave pública do Emissor para verificar se a assinatura contida no campo `signature` é válida para os bytes contidos no campo `payload`. Se a verificação criptográfica falhar, o token **DEVE** ser rejeitado imediatamente.
3. **Deserialização do Payload:**
   - Após a verificação da assinatura, o Validador **DEVE** deserializar os bytes do campo `payload` para uma instância da mensagem `SignetPayload`.
4. **Validação Temporal:**
   - O Validador **DEVE** verificar os claims temporais:
     - O timestamp atual **DEVE** ser anterior ao valor do campo `exp`.
     - O timestamp atual **DEVE** ser igual ou posterior ao valor do campo `iat`.
5. **Validação da Audiência:**
   - Se o campo `aud` estiver presente, o Validador **DEVE** verificar se sua própria identidade está contida na audiência declarada.
6. **Validação de Perfil (se aplicável):**
   - Se o Validador operar em um modo que exige revogação, ele **DEVE** executar a validação do perfil STATEFUL (ver Seção 8).

---

# 8. Perfis de Operação (`TokenProfile`)

A especificação define dois perfis para permitir flexibilidade entre escalabilidade e controle de sessão.

## 8.1. Perfil STATELESS

Este é o perfil padrão. Ele não requer consulta a um estado externo, maximizando a performance. Um token é considerado **STATELESS** se o campo `sid` do `SignetPayload` estiver vazio. A validação se baseia unicamente na assinatura e nos claims contidos no próprio token.

## 8.2. Perfil STATEFUL

Este perfil habilita a capacidade de revogação de tokens individuais.

- Um token é considerado **STATEFUL** se o campo `sid` do `SignetPayload` contiver um valor. Este valor **DEVE** ser um identificador único.
- Durante a validação de um token STATEFUL, o Validador, após completar todas as outras checagens, **DEVE** verificar o valor do `sid` contra um sistema de estado de revogação. Se o `sid` estiver na lista de revogados, o token **DEVE** ser rejeitado com um erro indicando que a sessão não é mais válida.

---

# 9. Algoritmo Criptográfico

A especificação Signet v1.0 **REQUER** o uso exclusivo do algoritmo de assinatura **Ed25519**. Implementações que utilizem outros algoritmos não são consideradas compatíveis com a v1.0.

---

# 10. Transporte em gRPC

O método **RECOMENDADO** para transportar um `SignetToken` serializado em uma chamada gRPC é através de metadados (headers).

- O nome do header **DEVERIA** ser `authorization-bin`.
- O uso do sufixo `-bin` é crucial, pois informa ao framework gRPC que os dados são binários, evitando codificações ineficientes e desnecessárias.

---

## 11. Considerações de Implementação e Segurança

Uma especificação define as regras, mas uma implementação segura requer a consciência dos riscos inerentes e das melhores práticas operacionais. Esta seção serve como um guia para os implementadores do Signet.

### 11.1. Gestão de Chaves Criptográficas

A segurança de um `SignetToken` depende da correta gestão das chaves de assinatura. A partir da v1.0, a especificação RECOMENDA fortemente o uso do padrão **KeyResolver** para validação de tokens em produção.

- **KeyResolver:** Um KeyResolver é uma função (ou interface) que, dado um `kid` (Key ID), retorna a chave pública correspondente para validação da assinatura. A API de validação DEVE aceitar um KeyResolver como argumento, e NÃO uma chave estática. Isso permite rotação transparente de chaves e suporte a múltiplos emissores.

#### Implementando um KeyResolver Seguro

- **Cache com TTL:** O KeyResolver DEVE implementar um cache com TTL (Time-To-Live) para as chaves públicas, minimizando consultas externas e mitigando ataques de negação de serviço (DoS) por requisições de chaves não existentes.
- **Retrocompatibilidade (`kid` vazio):** Se o campo `kid` estiver ausente ou vazio, o KeyResolver DEVE retornar a chave padrão configurada para o emissor, garantindo compatibilidade com tokens antigos ou ambientes de chave única.
- **Erro formal para `kid` desconhecido:** Se o `kid` informado não for encontrado, o KeyResolver DEVE retornar um erro formal, como `ErrUnknownKeyID`, e a validação do token DEVE falhar explicitamente.

- **Proteção da Chave Privada:** A chave privada do Emissor **NUNCA DEVE** ser partilhada ou exposta. **RECOMENDA-SE** o uso de um Hardware Security Module (HSM), como o Cloud HSM da GCP ou o AWS KMS, para gerar, armazenar e usar a chave sem que esta alguma vez saia do hardware seguro.
- **Distribuição da Chave Pública:** A distribuição segura da chave pública do Emissor para os Validadores é fundamental. Soluções como um endpoint de metadados seguro (semelhante a um JWKS endpoint), ou mecanismos de descoberta de confiança em uma service mesh (ex: SPIFFE/SPIRE), são **RECOMENDADOS**.
- **Rotação de Chaves:** Os implementadores **DEVERIAM** ter um processo para rotacionar as chaves de assinatura periodicamente, para limitar o impacto de uma eventual exposição de chave.

### 11.2. O Custo de Desempenho do Perfil STATEFUL

O perfil STATEFUL oferece a poderosa capacidade de revogação, mas esta funcionalidade tem um custo de desempenho que **DEVE** ser compreendido.

Cada validação de um token STATEFUL exige uma consulta de rede a um sistema de estado externo (ex: Redis, DynamoDB). Esta consulta introduz latência e um ponto de dependência adicional na sua arquitetura.

As equipes **DEVERIAM** avaliar cuidadosamente o trade-off entre a necessidade de revogação imediata e o impacto na latência e na complexidade da infraestrutura. O perfil STATELESS continua a ser a escolha recomendada para sistemas de altíssima performance que podem tolerar a validade de um token até à sua expiração natural.

### 11.3. Prevenção de Ataques de Repetição (Replay Attacks)

Em certos cenários, um atacante que capture um `SignetToken` válido poderia tentar "repeti-lo" para ganhar acesso não autorizado.

- **Tokens de Curta Duração:** A mitigação mais eficaz é o uso de tempos de expiração (`exp`) muito curtos (ex: de segundos a poucos minutos). Isso reduz drasticamente a janela de oportunidade para um ataque de repetição ser bem-sucedido.
- **Uso do `sid`:** No perfil STATEFUL, o `sid` pode ser usado para mitigar ataques de repetição. Uma vez que um token com um determinado `sid` é processado com sucesso, o Validador pode armazenar esse `sid` por um curto período de tempo e rejeitar quaisquer tentativas futuras de usar o mesmo `sid`.

### 11.4. Interoperabilidade e Contexto de Segurança

O Signet foi projetado para operar dentro de um ecossistema de segurança mais amplo, não isoladamente.

- **Coexistência com mTLS:** O Signet e o mTLS resolvem problemas diferentes e complementares. O mTLS autentica o serviço (transporte), enquanto o Signet autentica a requisição (aplicação). Eles **DEVEM** ser usados em conjunto para uma arquitetura Zero Trust robusta. A validação de um `SignetToken` só deve ocorrer após um handshake mTLS bem-sucedido.
- **Gateways de Tradução:** Em ambientes híbridos, um gateway que traduz um JWT (do frontend) para um `SignetToken` (para o backend) torna-se um componente de segurança crítico. Este gateway **DEVE** ser tratado como um Emissor de alta confiança, e a sua chave de assinatura deve ser protegida com o máximo rigor.