# 📚 Trabalho — Design Tático no DDD (Template para qualquer domínio)

> **Como usar:** copie este arquivo e substitua os **[colchetes]** com informações do **seu domínio** (e-commerce, marketplace, logística, educação, fintech, games, etc.).
> O objetivo é praticar Entidades, Value Objects, Agregados/AR, Repositórios e Eventos de Domínio — com foco em **invariantes** e **domínio rico**.

---

## 🧍Integrantes
1. Thayson Rodrigues de Souza - 361713
2. Guilherme Henrique de Amorin Sena - 362713
3. Ricardo Lourenço da Silva - 362046
4. Nícolas Garcia Maloucaze Pereira - 360381

---

## 🩺 1) Sobre o Domínio Escolhido
**Nome do domínio:** **Dispensador inteligente de medicação (VôLembrá)**  
**Objetivo do sistema:** **Assegurar que pacientes, especialmente idosos, tomem seus medicamentos na hora certa, com dispensação automática, alertas e notificações para responsáveis.**  
**Principais atores:** **Paciente, Responsável, Medicamento e Cuidador**  

---

## 🧩 2) Entidades vs Value Objects
Preencha a tabela justificando cada tipo (identidade vs. imutabilidade).

| Elemento | Tipo (Entidade/VO) | Por quê? (identidade/imutável) |
|---|---|---|
| **Paciente** | Entidade | Tem uma identidade única (ID) e um ciclo de vida próprio. |
| **Responsavel** | Entidade | Tem uma identidade única (ID) e um ciclo de vida próprio. |
| **Medicamento** | Entidade | Tem uma identidade única (ID) e um ciclo de vida próprio. |
| **LembreteDeMedicacao** | Entidade | 	Tem uma identidade única (ID) e representa a programação de uma dose. Seu estado muda ao longo do tempo. |
| **Horario** | Value Object | Representa um horário específico (por exemplo, 14:00). É imutável, e dois horários são iguais se tiverem o mesmo valor. |
| **Dose** | Value Object | Representa a quantidade e a unidade de um medicamento (por exemplo, 2 comprimidos). É imutável e sua igualdade é baseada no valor (quantidade e unidade). |

> Dica: Promova tipos semânticos: `Email`, `CPF/CNPJ`, `Money`, `IntervaloDeTempo`, `Endereco`, `Percentual`, `Quantidade`, etc. **VOs devem ser imutáveis** e com **igualdade por valor**.

---

## 🏗️ 3) Agregados e Aggregate Root (AR)
**Agregado Principal:** **LembreteDeMedicacao**  
**AR:** **LembreteDeMedicacao**  
**Conteúdo interno do agregado (apenas o necessário para consistência local):**  
- **Horario** (Value Object)
- **Dose** (Value Object)
- **Status** (Value Object/Enum)
- **TipoDeAlerta** (Value Object)

**Referências a outros agregados (por ID):**  
- **[PacienteId]** (referencia o agregado Paciente)
- **[MedicamentoId]** (referencia o agregado Medicamento)
- **[ResponsavelId]** (referencia o agregado Responsavel)

---

## 🧭 4) Invariantes e Máquina de Estados
Liste invariantes (devem ser verdadeiras ao final de cada transação).

**Invariantes:**
- **Um LembreteDeMedicacao deve ser associado a um Paciente existente.**
- **Um LembreteDeMedicacao deve ser associado a um Medicamento existente.**
- **Apenas um Paciente (dono do lembrete), Responsavel e o dispositivo (hardware) pode alterar o status do lembrete para 'Tomado', 'EM ATRASO' ou 'Ignorado'.**
- **O status de um LembreteDeMedicacao não pode retroceder (por exemplo, de 'Tomado' para 'Pendente').**

**Estados e transições da AR LembreteDeMedicacao:**
```
[EstadoInicial] -> [Pendente] -> [Tomado]
[EstadoInicial] -> [Pendente] -> [Em atraso]
[EstadoInicial] -> [Pendente] -> [Não tomado]
Regras:
- A transição para o estado 'Tomado' só é permitida se o lembrete estiver Pendente e a medicação for registrada no horário correto.
- A transição para o estado 'Em atraso' só é permitida se o lembrete estiver Pendente e a medicação for registrada 15 minutos após o horário correto.
- A transição para o estado 'Não tomado' só é permitida se o lembrete estiver Pendente e a medicação não for retirada no dispositivo após 1 hora.
```

---

## 🗃️ 5) Repositório do Agregado (interface)
> Repositório trabalha **apenas com a AR**, sem expor entidades internas do agregado. Consultas analíticas ficam fora (read models).

**Linguagem livre** (ex.: C#, Java, Kotlin, TS). Exemplo (C# assíncrono, adapte nomes):
```csharp
public interface ILembreteDeMedicacaoRepository
{
    Task<LembreteDeMedicacao?> ObterPorIdAsync(Guid id, CancellationToken ct = default);
    Task AdicionarAsync(LembreteDeMedicacao lembrete, CancellationToken ct = default);
    Task SalvarAsync(LembreteDeMedicacao lembrete, CancellationToken ct = default);
}
```

---

## 📣 6) Eventos de Domínio
Defina **2–4 eventos** com **payload mínimo** e **momento de publicação** (preferir **pós-commit**). Diferencie **evento interno** vs **evento de integração**.

| Evento | Quando ocorre | Payload mínimo | Interno/Integração | Observações |
|---|---|---|---|---|
| **LembreteDeMedicacaoTomado** | Ao confirmar a tomada do medicamento | LembreteId, PacienteId, MomentoDaTomada | Interno | Pode ser consumido para atualizar um histórico de doses ou gerar um alerta ao responsável. |
| **LembreteDeMedicacaoNaoTomado** | Ao ignorar o lembrete | LembreteId, PacienteId | Interno | Pode ser consumido para alertar o responsável ou registrar um evento de não-conformidade |
| **NovoLembreteCadastrado** | Ao cadastrar um novo lembrete | LembreteId, PacienteId, ResponsavelId, Horario | Interno | Dispara a lógica de agendamento de notificação. |

---

## 🗺️ 7) Diagrama (Mermaid ou ferramenta à sua escolha)
> Mostre **Agregados/AR**, **VOs** e **relacionamentos por ID** entre agregados (não “contenha” outros agregados).

**Exemplo de esqueleto Mermaid:**
```mermaid
classDiagram
  class LembreteDeMedicacao {
    +Guid Id
    +Guid PacienteId
    +Guid MedicamentoId
    +Dose Dose
    +DiaSemana DiaSemana
    +Horario Horario
    +Date DataFim
    +Agendar()
  }

  class Paciente {
    +Guid Id
  }

  class Medicamento {
    +Guid Id
  }

  class DiaSemana {
    +string Valor
  }

  class Horario {
    +TimeSpan Valor
  }

  class Dose {
    +double Quantidade
    +string Unidade
  }

  LembreteDeMedicacao --> Paciente : por Id
  LembreteDeMedicacao --> Medicamento : por Id
  LembreteDeMedicacao --> DiaSemana
  LembreteDeMedicacao --> Horario
  LembreteDeMedicacao --> Dose
```

---

## ✅ Checklist de Aceitação
- [X] **VOs imutáveis** e com **igualdade por valor** (nada de “string de CPF/Email”).
- [X] **Boundary do agregado** pequeno e com **invariantes claras**.
- [X] **Domínio rico**: operações do negócio como métodos (evitar `set` aberto).
- [X] **Repositório** focado na **AR** (sem `IQueryable`/detalhes de ORM no domínio).


## 📤 Entrega

- **Inclua**: link/imagem do **diagrama** + todas as seções acima preenchidas.
---

