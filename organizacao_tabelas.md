# **Organização de Tabelas no Projeto LifeLoom**

Este documento descreve as decisões arquiteturais relacionadas à organização das tabelas no banco de dados do projeto **LifeLoom**, justificando a estrutura adotada para atender aos requisitos do sistema. O projeto tem como foco a gestão de doadores, receptores e órgãos disponíveis para doação.

---

## **Objetivo do Documento**
- Explicar a estrutura das tabelas e seus relacionamentos.
- Justificar as decisões tomadas com base nos requisitos funcionais e não funcionais.
- Facilitar a manutenção e expansão do sistema.

---

## **Entidades e Relacionamentos**
![LifeLoom](https://github.com/user-attachments/assets/54ad76d6-6c1a-4cdb-b9ba-58b601d788d2)

### 1. **User**
**Por que foi separado?**  
A tabela `users` centraliza informações de doadores e receptores, permitindo flexibilidade e escalabilidade. Usamos o atributo `profile_id` para diferenciar perfis (ex.: doador, receptor, administrador). Essa abordagem:
- Reduz duplicação de dados (nome, e-mail, CPF).
- Permite a criação de funcionalidades específicas para cada perfil.

**Campos principais:**
- `name`, `email`, `cpf`, `birth_date`, `gender`: Dados demográficos do usuário.
- `profile_id`: Relaciona-se com a tabela `profiles`, indicando o tipo de usuário.
- Relacionamentos:
  - **Address:** Cada usuário pode ter um único endereço.
  - **Organs:** Relaciona os órgãos que um usuário pode doar ou receber.

---

### 2. **Organ**
**Motivação arquitetural:**  
A tabela `organs` é o núcleo do sistema. Ela armazena detalhes sobre órgãos disponíveis, sejam para doação ou recepção. Diferenciamos um órgão como doado ou recebido pelos campos `donor_id` e `recipient_id`. Essa abordagem:
- Garante que um órgão seja associado a um único doador ou receptor.
- Mantém flexibilidade para rastrear o ciclo de vida de cada órgão.

**Campos principais:**
- `organ_type_id`: Relaciona-se com a tabela `organ_types`, especificando o tipo do órgão (ex.: coração, fígado).
- `status`: Representa o estado atual do órgão (ex.: `Waiting`, `Matched`, `Donated`).
- `donor_id` e `recipient_id`: Relacionam o órgão aos usuários doadores e receptores.
- `expiration_date`: Ajuda a gerenciar a validade do órgão, essencial para a integridade do sistema.

---

### 3. **OrganType**
**Por que separar os tipos de órgãos?**  
A tabela `organ_types` evita a repetição de dados descritivos e regras específicas para cada tipo de órgão, como critérios de compatibilidade e tempo de preservação. Assim:
- As alterações em critérios de um órgão não afetam diretamente os registros na tabela `organs`.
- Mantém o sistema extensível para novos tipos de órgãos.

**Campos principais:**
- `name` e `description`: Descrevem o órgão.
- `compatibility_criteria`: JSON com critérios como faixa etária ou tipo sanguíneo.
- `default_preservation_time_minutes`: Tempo padrão para preservação.

---

### 4. **Profile**
**Justificativa:**  
A tabela `profiles` permite a diferenciação clara entre tipos de usuários (ex.: `Doador`, `Receptor`, `Administrador`). Esta separação:
- Facilita o uso de middlewares e políticas para controle de acesso com base no perfil.
- Torna o sistema adaptável para novos tipos de perfis.

---

### 5. **Address**
**Motivação:**  
A tabela `addresses` foi criada para normalizar os dados de localização dos usuários, garantindo:
- Reutilização eficiente dos dados de endereço.
- Compatibilidade com futuras funcionalidades que dependam de geolocalização, como limites de transporte de órgãos.

---

## **Decisões Arquiteturais**

### **1. Uso de Relacionamentos**
- Relacionamentos como `HasMany` (ex.: `users` para `organs`) e `BelongsTo` (ex.: `organs` para `users`) garantem:
  - Flexibilidade no gerenciamento de associações.
  - Performance otimizada em consultas.

### **2. Campos como Chaves Estrangeiras**
- O uso de chaves estrangeiras (`donor_id`, `recipient_id`, `organ_type_id`) permite:
  - Integridade referencial no banco de dados.
  - Consultas rápidas e associativas entre entidades.

### **3. JSON para Critérios de Compatibilidade**
- Armazenar critérios de compatibilidade no campo `compatibility_criteria` como JSON na tabela `organ_types`:
  - Evita complexidade no modelo relacional.
  - Garante que diferentes tipos de órgãos possam ter critérios personalizados.

### **4. Design Baseado em Extensibilidade**
A estrutura do banco de dados foi projetada para:
- Adicionar novos tipos de órgãos sem impacto nos dados existentes.
- Permitir novos perfis de usuário com ajustes mínimos.

---

## **Vantagens da Estrutura Adotada**

1. **Escalabilidade**
   - Adição de novos tipos de órgãos ou perfis de usuário sem grandes refatorações.
2. **Manutenibilidade**
   - Relacionamentos claros e modularidade reduzem a complexidade da manutenção.
3. **Flexibilidade**
   - Suporte a cenários complexos, como múltiplos perfis e diferentes estados de órgãos.

---

## **Cenários de Uso**

### Cenário 1: Cadastro de Órgãos
- O sistema utiliza `organ_type_id` para associar um órgão ao seu tipo.
- Os dados de preservação e compatibilidade são carregados dinamicamente a partir da tabela `organ_types`.

### Cenário 2: Gestão de Perfis
- O middleware `ValidateProfile` utiliza `profile_id` para autorizar acessos às rotas específicas.

### Cenário 3: Rastreabilidade de Órgãos
- O ciclo de vida de cada órgão é rastreado com base nos campos `status`, `expiration_date` e associações com `donor_id` e `recipient_id`.

---

## **Possíveis Melhorias Futuras**
1. **Histórico de Alterações**
   - Criar uma tabela de log para rastrear alterações nos dados de órgãos.
2. **Suporte a Localização**
   - Integração com APIs de geolocalização para calcular distâncias e sugerir hospitais próximos.
3. **Otimização de Consultas**
   - Indexação de campos frequentemente utilizados, como `status` e `expiration_date`.
