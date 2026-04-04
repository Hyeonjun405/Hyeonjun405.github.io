---
title: 06 Agent 패턴
date: 2026-04-04 10:00:00 +09:00
categories: [Sparta, SpartaWIL04]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - 어우 복잡해
   - 기본적으로 reAct는 사용하는 것으로 보는게 유리할듯
   - 어차피 반복적으로 사용하는게 기본 베이스
   - 이떄 이전에 질문한걸 활용해야하니 어차피 reAct패턴과 흡사
 - 결과적으로는 그냥 Agentic을 구현하는 방법론
   - 해석하고 판단하는걸 LLM에게 한번에 맡기지말고 
   - 순차적으로 판단할 수 있도록 유도하는 방법

## 2. Agent와 Agentic
### 1. Memo
 - 무언가를 대신 수행하는 프로그램 단위가 Agent
 - LLM이 어떤 순서로 판단하고 행동할지를 우리가 설계한 것이 Agentic한것
 - 그러나 본질적으로는 LLM을 호출하는 것

### 1. Agentic
 - “Agent”는 구조(개체),  
 - Agent가 가진 요소
   - 목표 (Goal)
   - 입력 (User 요청, 환경 데이터)
   - 판단 (LLM 추론)
   - 행동 (툴 호출, API 호출 등) 
 
### 2. Agentic
 - “Agentic”은 행동 방식(성향/패턴)
 - Agentic이 가진 요소
   - 스스로 계획 세움 (Planning)
   - 여러 단계로 문제 해결 (Multi-step reasoning)
   - 상황에 따라 행동 수정 (Adaptive)
   - 반복 실행 (Iteration / Loop)

### 3. Agentic 패턴

| 패턴           | 핵심 특징    | 언제 쓰냐      | 흐름주도              |
| ------------ | -------- | ---------- |-------------------|
| ReAct        | 생각+행동 반복 | Tool 기반 작업 | LLM이 매 스텝 결정      |
| Plan-and-Execute | 계획 먼저    | 단계 명확할 때   | LLM이 초반에 전체 결정    |
| Self-Reflection    | 자기 검증    | 품질 중요할 때   | LLM이 자기 자신을 검토    |
| Multi-Agent  | 역할 분리    | 복잡한 작업     | 여러 LLM이 역할 분담     |

## 3. 도구(Tools) 사용 & 기본 패턴
### 1.ReAct Agent
```
public String solveWithAgent(String goal) {
    log.info("Standard ReAct Agent 실행: {}", goal);

    String systemPrompt = """
        당신은 자율적인 AI 에이전트입니다. 목표 달성을 위해 다음 단계를 반복하세요:
        1. Thought: 현재 상황을 분석하고 필요한 행동을 생각합니다.
        2. Action: 적절한 도구를 선택하여 실행합니다.
        3. Observation: 도구의 결과를 확인하고 지식을 업데이트합니다.
        4. Answer: 모든 정보가 모이면 최종 답변을 작성합니다.
        """;

    return chatClientBuilder.build().prompt()
            .system(systemPrompt)
            .functions("getWeather", "calculator", "getCurrentTime") // 사용할 기능 등록
            .user(goal)
            .call()
            .content();
}
```

### 2. Plan-and-Execute (계획 후 실행)
```
public String planAndExecute(String goal) {
    log.info("Plan-and-Execute 시작: {}", goal);

    // 1. 계획 수립 (Planner)
    String plan = chatClientBuilder.build().prompt()
            .system("당신은 복잡한 목표를 위한 논리적인 단계를 수립하는 전략가입니다.")
            .user("목표: " + goal + "\n이 목표를 달성하기 위한 상세 계획을 번호를 매겨 세워주세요.")
            .call()
            .content();

    log.info("생성된 계획: \n{}", plan);

    // 2. 계획 실행 (Executor)
    return chatClientBuilder.build().prompt()
            .system("당신은 주어진 계획을 정확히 이행하는 실행 전문가입니다. 제공된 도구를 활용하세요.")
            .functions("getWeather", "calculator", "getCurrentTime")
            .user("수립된 계획: " + plan + "\n위 계획을 실행하고 최종 결과를 보고하세요.")
            .call()
            .content();
}
```

### 3. Self-Reflection(자기 반성)
```
public String solveWithReflection(String problem, int maxIterations) {
  log.info("Self-Reflection Agent 시작: {}", problem);
  
  ChatClient client = chatClientBuilder.build();
  String currentResponse = "";
  String feedback = "초기 시도";

  for (int i = 0; i < maxIterations; i++) {
      log.info("반복 단계: {}/{}", i + 1, maxIterations);

      // 1. 해결 시도 또는 개선
      currentResponse = client.prompt()
              .system("도구를 사용하여 문제를 해결하세요. 이전 피드백이 있다면 반영하여 답변을 개선하세요.")
              .functions("getWeather", "calculator", "getCurrentTime")
              .user("문제: " + problem + "\n이전 피드백: " + feedback)
              .call()
              .content();

      // 2. 검증 (Critique)
      feedback = client.prompt()
              .system("제시된 답변의 정확성과 논리적 결함을 검토하세요. 완벽하면 'APPROVED'라고 답하세요.")
              .user("검토 대상 답변: " + currentResponse)
              .call()
              .content();

      if (feedback.contains("APPROVED")) {
          log.info("답변 최종 승인됨");
          break;
      }
      log.warn("개선 필요 사항 발견: {}", feedback);
  }
  return currentResponse;
}
```

### 4. 도구제한
```
public String solveWithLimitedTools(String goal, String... toolNames) {
    return chatClientBuilder.build().prompt()
            .system("허용된 도구만 사용하여 문제를 해결하세요.")
            .functions(toolNames)
            .user(goal)
            .call()
            .content();
}
```

## 4. reAct 패턴
### 1. ReAct 패턴
- ReAct = Reason(생각) + Act(행동)을 반복하는 루프 구조

### 2. 주요 특징
- Thought (추론): 현재 상황을 분석하고, 목표 달성을 위해 '무엇을 해야 할지' 스스로 판단
- Action (행동): 판단에 따라 특정 도구(검색, DB 조회, Python 코드 실행 등)를 선택하고 실행
- Observation (관찰): 도구의 실행 결과(데이터)를 확인하고 학습
- Final Answer (최종 답변): 모든 정보가 수집되면 사용자에게 최종 결과를 전달

### 3. 흐름
```
 ※ 정해진 루프가 있는 상태에서 
 1. LLM 호출 
 2. LLM 호출 외 비즈니스 작업 
 3. 엔드포인트인지 확인 (for문 횟수 확인, 자기 검증, 엔티티 완성 등)
   - 종료포인트 O : 작업 종료 
   - 종료포인트 X : 해당 답변과 user 질문을 합쳐서 다시 LLM호출(1번 작업)
```

### 4. 소스
```
public String run(String userQuestion) {

    StringBuilder context = new StringBuilder(userQuestion).append("\n\n");

    for (int step = 0; step < MAX_STEPS; step++) {
        log.info("=== {}바퀴 ===", step + 1);

        String response = chatClient.prompt()
                .user(context.toString())
                .call()
                .content();

        log.info("[LLM 생성]\n{}", response);
        context.append(response).append("\n");

        // Final Answer 체크
        if (response.contains("Final Answer:")) {
            return extractValue(response, "Final Answer:");
        }

        // Action Input 파싱 → 툴 실행 → Observation 주입
        if (response.contains("Action Input:")) {
            String actionInput = extractValue(response, "Action Input:");
            String observation = searchTool.search(actionInput);

            log.info("[툴 실행] {}", observation);
            context.append("Observation: ").append(observation).append("\n\n");
        }
    }

    throw new IllegalStateException("MAX_STEPS(%d) 초과".formatted(MAX_STEPS));
}
```

## 5. 전략적 계획 및 단계별 실행 (Plan-and-Execute)
### 1. 프롬프트 상수/ private
```
//프롬프트 상수화
private static final String PLANNER_SYSTEM_PROMPT = """
  당신은 효율적인 계획을 수립하는 전문가입니다. 목표 달성을 위해 구체적이고 실행 가능한 계획을 세우세요.
  계획 작성 형식:
  1. [단계명]: [작업 내용] - 도구: [도구명]
  2. [단계명]: [작업 내용] - 도구: [도구명]
  
  사용 가능 도구: getWeather, calculator, getCurrentTime
  각 단계는 이전 단계의 결과를 활용하도록 설계하세요.
  """;
  
// 계획 수립  
private String createPlan(String goal) {
  return chatClientBuilder.build().prompt()
          .system(PLANNER_SYSTEM_PROMPT)
          .user(goal)
          .call()
          .content();
}

// 스탭 추출
private List<String> parseSteps(String plan) {
  return plan.lines()
          .map(String::trim)
          .filter(line -> line.matches("^\\d+\\..*")) // "1. 단계명" 형태만 추출
          .collect(Collectors.toList());
}

//결과 취합
private String summarizeResults(String goal, List<StepResult> stepResults) {
  String combined = stepResults.stream()
          .map(s -> "단계 " + s.stepNumber() + ": " + s.result())
          .collect(Collectors.joining("\n"));

  return chatClientBuilder.build().prompt()
          .system("실행 결과들을 종합하여 사용자에게 최종 보고서를 작성하세요.")
          .user("목표: " + goal + "\n결과들:\n" + combined)
          .call()
          .content();
}
```

### 2. 기본 Plan-and-Execute
```
public String executeWithPlanning(String goal) {
    log.info("=== Basic Plan-and-Execute 시작: {} ===", goal);

    String plan = createPlan(goal);
    log.info("수립된 계획:\n{}", plan);

    String executionPrompt = """
        다음 계획을 단계별로 실행하고 최종 결과를 요약하세요.
        계획: %s
        원래 목표: %s
        """.formatted(plan, goal);

    return chatClient.build().prompt()
            .system("당신은 계획을 정확하게 실행하는 AI 에이전트입니다.")
            .functions("getWeather", "calculator", "getCurrentTime")
            .user(executionPrompt)
            .call()
            .content();
}
```

### 3. 상세 추적형 실행 (Detailed Tracking)
```
public ExecutionResult executeWithDetailedTracking(String goal) {
    log.info("=== Detailed Tracking Agent 시작 ===");

    String plan = createPlan(goal);
    List<String> steps = parseSteps(plan);
    List<StepResult> stepResults = new ArrayList<>();
    String context = ""; // 이전 단계들의 누적 결과

    for (int i = 0; i < steps.size(); i++) {
        String currentStep = steps.get(i);
        int stepNum = i + 1;
        log.info("단계 {}/{} 실행: {}", stepNum, steps.size(), currentStep);

        String stepResult = chatClient.build().prompt()
                .system("당신은 주어진 단계를 실행하는 전문가입니다. 이전 문맥을 참고하세요.")
                .functions("getWeather", "calculator", "getCurrentTime")
                .user("전체목표: %s\n현재단계: %s\n이전까지의 문맥: %s".formatted(goal, currentStep, context))
                .call()
                .content();

        stepResults.add(new StepResult(stepNum, currentStep, stepResult));
        context += "\n[단계 " + stepNum + " 결과]: " + stepResult;
    }

    String finalSummary = summarizeResults(goal, stepResults);
    return new ExecutionResult(goal, plan, stepResults, finalSummary);
}
```

### 4. 적응형 계획
```
public String executeWithAdaptivePlanning(String goal, int maxReplanning) {
    log.info("=== Adaptive Planning Agent 시작 ===");
    
    String currentPlan = createPlan(goal);
    String lastResult = "";

    for (int i = 0; i <= maxReplanning; i++) {
        lastResult = executeWithPlanning(currentPlan);
        
        // 결과 검증
        String validation = chatClient.build().prompt()
                .system("결과가 목표를 달성했는지 평가하세요. 달성했다면 'SUCCESS', 아니면 'INCOMPLETE: 이유'라고 답하세요.")
                .user("목표: %s\n결과: %s".formatted(goal, lastResult))
                .call().content();

        if (validation.startsWith("SUCCESS")) {
            log.info("목표 달성 성공 (시도: {})", i + 1);
            return lastResult;
        }

        log.warn("계획 재수립 필요 (사유: {})", validation);
        currentPlan = chatClientBuilder.build().prompt()
                .system("피드백을 바탕으로 계획을 수정하세요.")
                .user("기존계획: %s\n실패결과: %s\n피드백: %s".formatted(currentPlan, lastResult, validation))
                .call().content();
    }
    return lastResult;
}
```

## 6. 협업형 멀티 에이전트
### 1. 상수 / 개별 프롬프트
```

// 연구원 에이전트
private static final String RESEARCHER_PROMPT = """
    당신은 정보 수집 전문 연구원입니다. 도구(getWeather, getCurrentTime 등)를 활용하여 
    문제 해결에 필요한 팩트와 데이터를 체계적으로 수집하세요.
    """;

  private String executeResearcher(String input) {
      return chatClientBuilder.build().prompt()
              .system(RESEARCHER_PROMPT)
              .functions("getWeather", "getCurrentTime") // 연구용 도구
              .user(input)
              .call().content();
  }

// 데이터분석가 에이전트
private static final String ANALYST_PROMPT = """
    당신은 데이터 분석 전문가입니다. 수집된 정보를 바탕으로 인사이트를 도출하세요.
    수치 계산이 필요하면 calculator 도구를 사용하고, 논리적인 인과관계를 분석하세요.
    """;

private String executeAnalyst(String input) {
    return chatClientBuilder.build().prompt()
            .system(ANALYST_PROMPT)
            .functions("calculator") // 분석용 도구
            .user(input)
            .call().content();
}

// 리포트 작성 에이전트
private static final String WRITER_PROMPT = """
    당신은 전문 리포트 작성자입니다. 분석 내용을 바탕으로 구조화된 보고서를 작성하세요.
    구성: 1.요약, 2.주요발견, 3.상세분석, 4.결론
    """;
    
private String executeWriter(String input) {
    return chatClientBuilder.build().prompt()
            .system(WRITER_PROMPT)
            .user(input)
            .call().content();
}
```

### 2. 순차적 협업 (Sequential Chain)
```
public String solveSequential(String problem) {
  log.info("순차적 협업 시작: {}", problem);

  //순서대로 한번씩 호출해서 사용함.
  String research = executeResearcher(problem);
  String analysis = executeAnalyst(research);
  return executeWriter(analysis);
}
```

### 3. 동적 파이프라인 (Dynamic Orchestration)
```
public String solveWithDynamicAgents(String problem) {
    String problemType = analyzeProblemType(problem);
    log.info("분석된 문제 유형: {}", problemType);

    //문제 타입에 따른 프롬프트 조합 선택
    List<AgentRole> pipeline = switch (problemType) {
        case "CALCULATION" -> List.of(AgentRole.ANALYST, AgentRole.WRITER);
        case "DATA_COLLECTION" -> List.of(AgentRole.RESEARCHER, AgentRole.WRITER);
        default -> List.of(AgentRole.RESEARCHER, AgentRole.ANALYST, AgentRole.WRITER);
    };

    //프롬프트 조합별 작업 진행
    String currentInput = problem;
    for (AgentRole role : pipeline) {
        currentInput = switch (role) {
            case RESEARCHER -> executeResearcher(currentInput);
            case ANALYST -> executeAnalyst(currentInput);
            case WRITER -> executeWriter(currentInput);
        };
    }
    return currentInput;
}

private String analyzeProblemType(String problem) {
    return chatClientBuilder.build().prompt()
            .system("문제 유형을 하나만 선택하세요: CALCULATION, DATA_COLLECTION, ANALYSIS")
            .user(problem)
            .call().content().trim();
}
```

### 4. 피드백 루프 (Iterative Review)
```
public String solveWithFeedback(String problem, int maxIterations) {
    String research = executeResearcher(problem);
    String result = "";

    for (int i = 0; i < maxIterations; i++) {
        String analysis = executeAnalyst(research);
        
        // 검토자(Reviewer) 에이전트 호출
        String feedback = chatClientBuilder.build().prompt()
                .system("분석 결과가 충분한지 평가하세요. 완벽하면 'APPROVED', 부족하면 피드백을 주세요.")
                .user("문제: " + problem + "\n분석결과: " + analysis)
                .call().content();

        if (feedback.contains("APPROVED")) {
            return executeWriter(analysis);
        }
        
        log.warn("품질 미달로 재연구 수행 (반복: {}): {}", i + 1, feedback);
        research = executeResearcher("이전 연구: " + research + "\n피드백 반영: " + feedback);
    }
    return result;
}
```
