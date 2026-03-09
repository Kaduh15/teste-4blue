# Automação de Matrículas e Boas-vindas

Este projeto foi desenvolvido como parte de um desafio técnico.  
A proposta é criar uma automação utilizando **n8n** para processar matrículas de novos alunos.

O fluxo recebe os dados via webhook, trata e normaliza as informações, salva no banco de dados e envia um e-mail de confirmação para o aluno.

---

## Fluxo da automação

O workflow segue as seguintes etapas:

1. Recebimento dos dados via **Webhook (POST)**
2. Validação dos dados recebidos
3. Normalização das informações
4. Conversão da data de nascimento
5. Verificação do tipo de curso
6. Inserção no banco PostgreSQL
7. Envio de e-mail de confirmação

Fluxo resumido:

```

Webhook → Validação → Normalização → Switch Curso → PostgreSQL → Email

```

---

## Vídeo explicando o fluxo

Link do vídeo:
  [Video do fluxo](https://drive.google.com/file/d/1FgRdyD6MAnJa2zXPHxxHLmT2S4zjcWeP/view?usp=sharing)

---

## Tratamento dos dados

Durante o fluxo os dados passam por algumas transformações.

### Nome

A primeira letra de cada nome é convertida para maiúscula.

Exemplo:

```

john smith → John Smith

```

### Telefone

São removidos caracteres especiais utilizando `.replace()` / regex.

Exemplo:

```

(11) 99999-9999 → 11999999999

```

### Data de nascimento

A data é convertida do formato brasileiro para o formato americano.

```

28/02/2020 → 2020-02-28

```

---

## Lógica de negócio

O fluxo utiliza um **Switch Node** para direcionar os dados para a tabela correta.

```

Tech → matriculas_tech
Business → matriculas_biz

```

---

## Banco de dados

Os dados tratados são inseridos em um banco **PostgreSQL** nas tabelas:

- `matriculas_tech`
- `matriculas_biz`

---

## Envio de e-mail

Após a matrícula ser registrada no banco, o fluxo envia um e-mail de confirmação para o aluno.

Exemplo de mensagem:

```

Olá João Silva, sua matrícula no curso Tech foi recebida com sucesso!

```

---

## Desafio oculto identificado

Um possível problema nesse fluxo é a **duplicação de matrículas**.

Webhooks podem ser enviados mais de uma vez (por retry, erro de rede ou envio duplicado), o que poderia gerar múltiplos registros do mesmo aluno.

Para evitar esse problema foi adicionada uma etapa de verificação no banco antes da inserção.

Fluxo aplicado:

```

Webhook
↓
Verificar se email já existe
↓
IF
├─ Já existe → interrompe o fluxo
└─ Não existe → continua para inserção

````

Isso evita registros duplicados e possíveis problemas operacionais.

---

## Exemplo de payload recebido

```json
{
  "nome_completo": "john smith",
  "email": "john@4blue.com.br",
  "data_nascimento": "28/02/2020",
  "telefone": "(11) 99999-9999",
  "curso": "Tech"
}
````

---

## Autor

Carlos Eduardo (Kadu)
