1# Chapter 5. Implementing Simple Business Logic

## 핵심 개념 (질문에 답하며 채워보세요)

### 비즈니스 로직 구현 패턴 개요
- 왜 비즈니스 로직이 소프트웨어에서 가장 중요한 부분이라고 책이 말하나요? UI가 멋지고 DB가 빨라도 왜 그것만으로는 부족한가요?
  - 비즈니스 로직은 소프트웨어가 만들어지는 이유 자체이기 때문이다. UI가 멋지고 DB가 빨라도, 소프트웨어가 비즈니스에 쓸모없다면 값비싼 기술 데모일 뿐이다.
- 2장에서 배운 "서브도메인마다 전략적 중요도와 복잡도가 다르다"는 전제가, 왜 이 챕터(그리고 다음 챕터들)에서 "서브도메인마다 다른 구현 패턴을 쓴다"는 결론으로 이어질까요?
  - 서브도메인마다 전략적 중요도와 복잡도가 다르다면, 모든 서브도메인에 똑같은 구현 패턴을 쓰는 건 비효율적이다. 단순한 로직에 너무 정교한 패턴을 쓰면 불필요한(accidental) 복잡도가 생기고, 복잡한 로직에 너무 단순한 패턴을 쓰면 로직이 여기저기 중복되고 일관성이 깨진다. 그래서 복잡도 수준에 맞는 패턴을 골라 써야 한다는 결론으로 이어지며, 이 챕터는 단순한 로직용 패턴(Transaction Script, Active Record)부터 시작해 다음 챕터들에서 점점 복잡한 로직용 패턴(Domain Model 등)으로 나아간다.    

### 트랜잭션 스크립트 (Transaction Script)
- Martin Fowler의 정의("Organizes business logic by procedures where each procedure handles a single request from the presentation")에서, 이 패턴은 시스템의 public interface를 무엇의 모음으로 보나요?
  - 소비자(consumer)가 실행할 수 있는 비즈니스 트랜잭션(business transactions)들의 모음으로 본다.
- 이 패턴에서 각 프로시저가 반드시 충족해야 할 단 하나의 요구사항은 무엇인가요? (힌트: 패턴 이름 자체에 그 답이 들어 있어요)
  - 트랜잭션 동작(transactional behavior)이다. 각 연산은 성공하거나 실패해야 하며, 절대 유효하지 않은 상태(invalid state)로 끝나서는 안 된다.
- 트랜잭션이 가장 곤란한 시점에 실패해도 시스템은 어떤 상태를 유지해야 하나요? 그걸 보장하는 두 가지 방법은 무엇인가요? (하나는 변경을 되돌리는 것, 다른 하나는 ○○ 조치를 실행하는 것)
  - 일관된 상태(consistent state)를 유지해야 한다. ① 실패 시점까지 만들어진 변경을 롤백(rollback)하거나, ② 보상 조치(compensating actions)를 실행해서 보장한다.

### 트랜잭션 스크립트 (Transaction Script): "It's Not That Easy!"
- 저자는 이 패턴이 단순해 보이지만 "가장 잘못 구현하기 쉬운 패턴"이라고 말해요. 왜 그렇다고 할까요? (힌트: 저자가 디버깅한 프로덕션 이슈의 상당수가 어디서 비롯됐다고 하나요?)
  - 저자가 디버깅하고 고친 수많은 프로덕션 이슈들이, 결국 시스템 비즈니스 로직의 트랜잭션적 동작을 잘못 구현(misimplementation)한 데서 비롯됐기 때문이다. 패턴 자체는 단순해 보이지만, 그 단순함 때문에 트랜잭션 보장을 놓치기 쉽다.
- **Lack of transactional behavior** 예시 — `UPDATE Users` 다음에 `INSERT INTO VisitsLog`를 별도로 실행하는 코드에서, 7번째 줄(UPDATE) 이후 9번째 줄(INSERT) 이전에 장애가 나면 어떤 불일치 상태가 생기나요? 이걸 고치는 가장 직접적인 방법은 무엇인가요?
  - Users 테이블만 업데이트되고 VisitsLog 테이블에는 대응하는 레코드가 기록되지 않는 불일치 상태가 생긴다. 가장 직접적인 해결책은 두 데이터 변경을 모두 포함하는 트랜잭션을 만드는 것이다(시작 시 `StartTransaction`, 실패 시 `Rollback`, 성공 시 `Commit`).
- **분산 트랜잭션 (Distributed transactions)** 예시 — DB를 업데이트한 다음 메시지 버스에 publish하는 코드에서는, 똑같은 종류의 문제인데 왜 "쉽게 고쳐지지 않는다"고 할까요? (힌트: 두 개의 서로 다른 저장 메커니즘을 하나의 트랜잭션으로 묶을 수 있나요?)
  - DB와 메시지 버스는 서로 다른 저장 메커니즘이라 하나의 트랜잭션으로 묶을 수 없기 때문이다. 여러 저장 메커니즘에 걸친 분산 트랜잭션은 복잡하고, 확장하기 어렵고, 오류가 나기 쉬워서 보통은 피한다. (8장의 CQRS, 9장의 outbox 패턴이 이 문제를 다룬다.)
- **암시적 분산 트랜잭션 (Implicit distributed transactions)** 예시(`UPDATE Users SET visits=visits+1`) — 이 메서드는 단일 테이블의 값 하나만 바꾸는데, 왜 이것도 "분산 트랜잭션"이라고 부를 수 있나요? (힌트: `void` 메서드라도 호출자에게 무언가를 "통신"하고 있어요. 무엇을요?)
  - 메서드가 `void`라도 연산이 성공했는지 실패했는지를 호출자(caller)에게 통신(communicate)한다 — 실패하면 호출자가 예외(exception)를 받는다. 즉 이 연산은 데이터베이스와, 메서드를 호출한 외부 프로세스 양쪽에 정보를 전달하는 셈이라, 저장소가 하나뿐이어도 "분산 트랜잭션"으로 볼 수 있다.
- 위 예시에서, 호출자가 (성공 여부를 통신받지 못해) 실패로 간주하고 같은 작업을 재시도하면 카운터는 몇 번 증가하게 되나요? 이 문제를 해결하는 두 가지 방법은 각각 어떻게 동작하나요? (하나는 작업을 멱등하게 만드는 것, 다른 하나는 ○○ 동시성 제어)
  - 카운터는 의도한 1번이 아니라 2번 증가하게 된다. ① 작업을 멱등(idempotent)하게 만든다 — 호출자가 현재 값을 먼저 읽어서 1 증가시킨 최종 값을 파라미터로 넘기고, `LogVisit`은 그 값을 그대로 저장한다. 같은 요청이 여러 번 실행돼도 결과가 같다. ② 낙관적 동시성 제어(optimistic concurrency control)를 쓴다 — 호출자가 호출 전에 읽은 현재 값을 파라미터로 넘기고, `LogVisit`은 그 값이 DB의 현재 값과 같은지 `WHERE` 조건으로 확인한 뒤에만 업데이트한다. 재시도 시에는 값이 이미 바뀌어 있어 조건이 충족되지 않으므로 중복 증가가 막힌다.

### Transaction Script: 언제 쓰는가
- 이 패턴이 자연스럽게 잘 맞는 서브도메인의 종류와 사용 사례는 무엇인가요? (힌트: ETL, 그리고 외부 시스템 통합 시 어떤 레이어의 일부로 쓰일 수 있나요?)
  - 정의상 비즈니스 로직이 단순한 지원 하위도메인(supporting subdomain)에 자연스럽게 맞는다. 사용 사례로는 ETL(extract-transform-load) 작업, 외부 시스템 통합 시 일반 하위 도메인과의 어댑터, 충돌 방지 계층(Anticorruption Layer)의 일부가 있다.
- 이 패턴의 가장 큰 장점(단순함)이 동시에 단점이 되는 이유는 무엇인가요? 그래서 어떤 종류의 서브도메인에는 절대 쓰면 안 된다고 할까요?
  - 비즈니스 로직이 복잡해질수록 여러 트랜잭션에 로직이 중복되기 쉽고, 그 중복된 코드가 서로 어긋나면(out of sync) 일관성 없는 동작으로 이어진다. 그래서 핵심 하위도메인(core subdomain)에는 절대 쓰면 안 된다 — 이 패턴은 핵심 하위도메인의 높은 복잡도를 감당하지 못한다.
- 책은 Transaction Script가 종종 "안티패턴"으로 취급받는다는 것을 인정하면서도, 왜 이 패턴이 여전히 중요하다고(이후에 배울 모든 패턴의 기반이라고) 말할까요?
  - 단순함 때문에 가끔 안티패턴으로 취급받기도 하지만, 이 챕터와 이후 챕터에서 다룰 비즈니스 로직 구현 패턴들이 모두 어떤 형태로든 트랜잭션 스크립트를 기반으로 하고 있기 때문에, 소프트웨어 개발에서 여전히 어디에나(ubiquitous) 쓰이는 핵심 토대다.

### Active Record
- Active Record는 Transaction Script와 어떤 점에서 비슷하고 어떤 점에서 다른가요? (힌트: 둘 다 "단순한" 비즈니스 로직을 다루지만, 다루는 데이터 구조의 복잡도가 달라요)
  - Transaction Script처럼 Active Record도 단순한 비즈니스 로직을 다루는 데 적합하다는 점은 같다. 다른 점은 다루는 데이터 구조다 — Transaction Script는 평평한(flat) 레코드를 가정하는 반면, Active Record는 일대다·다대다 관계를 가진 더 복잡한 객체 트리/계층 구조를 다룰 수 있다.
- 이 패턴의 이름이 "active"인 이유는 무엇인가요? (힌트: 데이터 구조 자체가 스스로 무엇을 구현하고 있나요?)
  - 각 데이터 구조 자체가 "활성(active)"이기 때문이다 — 즉, 데이터 구조 스스로 CRUD 같은 데이터 접근(data access) 로직을 구현하고 있다.
- Active Record 객체는 퍼시스턴스(CRUD) 책임 외에도 무엇을 포함할 수 있나요? 그럼에도 이 패턴의 "특징적인 성질(distinctive feature)"은 무엇이라고 할까요? (데이터와 행동이 어떻게 돼 있나요?)
  - 필드에 새로 할당되는 값을 검증(validate)하거나, 객체 데이터를 다루는 비즈니스 관련 절차를 구현하는 등 비즈니스 로직을 포함할 수 있다. 그럼에도 이 패턴의 특징적 성질은 데이터 구조와 행동(behavior, 비즈니스 로직)의 분리다 — 보통 필드에 public getter/setter가 있어서 외부 프로시저가 그 상태를 자유롭게 바꿀 수 있다.

### Active Record: 언제 쓰는가
- 이 패턴이 지원할 수 있는 비즈니스 로직의 복잡도 수준은 어느 정도인가요?
  - Active Record는 본질적으로 데이터베이스 접근을 최적화한 Transaction Script이기 때문에, 비교적 단순한 비즈니스 로직 — 사용자 입력을 검증하는 정도가 최대인 CRUD 연산 — 까지만 지원할 수 있다.
- 책은 Active Record가 "anemic domain model 안티패턴"이라고 비판받는다는 걸 알고 있으면서도, 왜 그 부정적 함의("anemic", "antipattern")에 동의하지 않을까요? (힌트: 패턴을 "도구"로 보는 관점 — 도구가 해를 끼치는 건 언제일까요?)
  - 저자는 패턴을 도구(tool)로 본다 — 도구는 잘못된 맥락에 쓰일 때만 해를 끼친다. 비즈니스 로직이 단순할 때 Active Record를 쓰는 건 전혀 문제가 없고, 오히려 단순한 로직에 더 정교한 패턴을 쓰면 불필요한 accidental complexity가 생겨 해가 된다. 그래서 "빈혈/안티패턴"이라는 부정적 함의에 동의하지 않는다.
- "Active Record"라는 이름이 가리키는 게 디자인 패턴인지, 특정 프레임워크(예: Rails의 Active Record)인지 — 책은 이 둘을 어떻게 구분하나요?
  - 이 책에서 "Active Record"는 디자인 패턴을 가리키는 것이며, (Rails 같은) Active Record 프레임워크를 가리키는 게 아니다. 패턴 이름은 Martin Fowler의 «Patterns of Enterprise Application Architecture»에서 처음 만들어졌고, 프레임워크는 그 패턴을 구현하는 한 가지 방법으로 나중에 등장한 것이다.

### Be Pragmatic
- 데이터 일관성을 다소 완화해도 괜찮을 수 있는 경우의 예시(초당 수십억 건의 IoT 이벤트 수집)에서, 책이 던지는 질문은 무엇인가요? 그 답을 결정하는 기준은 무엇일까요?
  - 책이 던지는 질문은 "이벤트의 0.001%가 중복되거나 손실돼도 정말 큰 문제인가?"이다. 답을 결정하는 기준은 보편적 법칙이 아니라, 그 비즈니스 도메인에서 그 정도 손실이 실제로 어떤 위험과 비즈니스적 영향(risks and business implications)을 미치는지를 평가하는 것이다.

### Conclusion
- 이번 챕터에서 다룬 두 패턴이 각각 가장 잘 맞는 서브도메인의 종류는 무엇인가요?
  - 둘 다 supporting subdomain에 가장 잘 맞는다 — Transaction Script는 ETL 같은 단순한 절차적 작업에, Active Record는 더 복잡한 데이터 구조를 다루는 supporting subdomain이나 generic subdomain 통합·모델 변환 작업에 적합하다.
- 다음 챕터에서는 어떤 종류의 비즈니스 로직(그리고 어떤 패턴)으로 넘어간다고 예고하나요?
  - 다음 챕터(6장)에서는 더 복잡한(complex) 비즈니스 로직으로 넘어가서, 그 복잡성을 다루는 domain model 패턴을 소개한다고 예고한다.

---

## 요약 (공부한 내용을 한 문장으로 요약해 보세요)

> 비즈니스 로직이 단순한 경우, Transaction Script는 트랜잭션적 동작(성공·실패 또는 롤백·보상 조치)을 보장하는 절차로 로직을 구조화하고, Active Record는 거기에 더해 더 복잡한 데이터 구조에 대한 CRUD 접근을 캡슐화하지만, 두 패턴 모두 단순한 로직(주로 supporting/generic subdomain)에만 적합하며 core subdomain에는 절대 쓰면 안 된다.

---

## 밑줄 친 문장 (가장 인상 깊었던 문장을 골라보세요)

> "This pattern is a tool. Like any tool, it can solve problems, but it can potentially introduce more harm than good when applied in the wrong context." (71쪽 — 원문 대조 확인됨)
>
> 이 패턴은 도구다. 다른 도구들처럼, 문제를 해결할 수도 있지만 잘못된 맥락에 적용되면 오히려 이득보다 해를 끼칠 수도 있다.

---

## 연습 문제 (Self-Check)

(이 네 문항은 책의 실제 챕터 말미 Exercises와 거의 동일해서, Appendix B의 정답을 기준으로 답함)

1. 핵심 서브도메인(core subdomain)의 비즈니스 로직을 구현할 때, 이 챕터에서 다룬 두 패턴(Transaction Script, Active Record) 중 어느 것을 써야 하나요?
   - C: 둘 다 쓸 수 없다. Transaction Script와 Active Record 모두 단순한 비즈니스 로직에 적합한 패턴인데, core subdomain은 그보다 더 복잡한 비즈니스 로직을 다루기 때문이다.
2. 다음 코드는 고수준 트랜잭션 메커니즘이 없다고 가정해요. 어떤 데이터 일관성 문제가 발생할 수 있나요?
   ```
   public void CreateTicket(TicketData data)
   {
       var agent = FindLeastBusyAgent();

       agent.ActiveTickets = agent.ActiveTickets + 1;
       agent.Save();

       var ticket = new Ticket();
       ticket.Id = Guid.New();
       ticket.Data = data;
       ticket.AssignedAgent = agent;
       ticket.Save();

       _alerts.Send(agent, "You have a new ticket!");
   }
   ```
   - D: 위 문제들이 전부 가능하다.
     - a. 6번째 줄(`agent.Save()`) 이후 실행이 실패하고 호출자가 재시도했는데 `FindLeastBusyAgent`가 같은 에이전트를 다시 고르면, 그 에이전트의 `ActiveTickets` 카운터가 1보다 더 많이 증가할 수 있다.
     - b. 6번째 줄 이후 실행이 실패했는데 호출자가 재시도하지 않으면, 카운터는 증가했지만 티켓 자체는 생성되지 않는다.
     - c. 12번째 줄(`ticket.Save()`) 이후 실행이 실패하면, 티켓은 생성·배정됐지만 14번째 줄의 알림은 전송되지 않는다.
3. 위 코드에는 한 가지 더 가능한 엣지 케이스가 있어요. 무엇일까요?
   - 12번째 줄(`ticket.Save()`) 이후 실행이 실패하고 호출자가 재시도해서 성공하면, 같은 티켓이 두 번 저장되고 배정된다(중복 생성).
4. 책 서문에 나온 가상의 회사 WolfDesk를 떠올려보면(고객지원 티켓 시스템), 그 시스템의 어떤 부분이 transaction script나 active record로 구현될 수 있을까요?
   - WolfDesk의 모든 supporting subdomain이 비즈니스 로직이 비교적 단순하기 때문에 좋은 후보다.
     - a. 테넌트(tenant)의 티켓 카테고리 관리
     - b. 고객이 지원 티켓을 열 수 있는 대상인, 테넌트의 제품 관리
     - c. 테넌트의 지원 에이전트 근무 일정 입력
