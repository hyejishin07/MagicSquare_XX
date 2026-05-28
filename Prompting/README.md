# Prompting — 대화형 프롬프트 Transcript

4×4 Magic Square 프로젝트의 **문제 정의 워크플로**를 재현할 수 있는 대화형 프롬프트 기록입니다.

## 문서 목록

| 파일 | 설명 |
|------|------|
| [interactive_prompt_transcript.md](./interactive_prompt_transcript.md) | Turn 0~7 USER/ASSISTANT transcript + Compact Prompt Chain |
| [prompt_chain.txt](./prompt_chain.txt) | 복사-붙여넣기 전용 순차 프롬프트 (구분선 포함) |

## 사용 방법

1. 새 AI 세션을 연다.
2. `prompt_chain.txt` 또는 transcript의 **Compact Prompt Chain**을 **위에서 아래로** 순서대로 붙여넣는다.
3. 각 STEP 응답을 확인한 뒤 다음 Block을 입력한다.

## 공통 규칙 (모든 STEP)

- 구현 설계 금지
- 코드 언급 금지
- 알고리즘 설명 금지
- 구조화된 Markdown 응답

## 관련 폴더

| 폴더 | 내용 |
|------|------|
| `Report/` | STEP 1~5 결과 보고서 |
| `Prompting/` | 본 transcript (프롬프트 재현용) |

## 작성 이력

| 날짜 | 내용 |
|------|------|
| 2026-05-28 | Turn 0~7 transcript 및 prompt_chain보내기 |
