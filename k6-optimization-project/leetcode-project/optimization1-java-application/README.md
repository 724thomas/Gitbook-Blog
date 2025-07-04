# Optimization1 - Java application

시스템이 기본적으로 기능 요구사항을 충족하는 것을 확인한 이후, 실제 서비스 환경에서의 성능 병목을 해결하고 응답 속도 및 처리 효율을 개선하기 위해 다양한 성능 테스트와 최적화를 진행하였습니다.&#x20;

각 주요 API에 대해 성능 병목의 원인을 가설로 설정하고, 구체적인 실험을 통해 이를 검증한 후 개선 작업을 수행하였습니다. 이 과정에서는 **페이징 전략 개선, 캐싱 도입, DB 인덱스 최적화, 멱등성 처리, Redis 구조 활용** 등 다양한 방식의 개선을 단계적으로 도입하였으며, 그에 따른 **쿼리/응답 시간 비교, 자원 사용량 감소, 시스템 안정성 향상** 등의 실질적인 효과를 수치 기반으로 분석하여 기록하였습니다.



[get-problemlist.md](get-problemlist.md "mention"): 커서 기반 페이지네이션

[get-problemdetail.md](get-problemdetail.md "mention"): Redis 캐싱

[post-submit-problem.md](post-submit-problem.md "mention"): 캐싱 & 멱등키 도입 시도

[get-leaderboard.md](get-leaderboard.md "mention"): Redis SortedSet 캐싱
