---
description: 유저모드와 커널 모드 구분 이유
---

# Reason for distinguishing between user mode and kernel mode

1. **보안**
   * 사용자 프로그램이 시스템 리소스에 직접 접근하는 것을 방지
   * ex) 사용자가 실행하는 일반적인 애플리케이션(예: 워드 프로세서나 웹 브라우저)이 시스템의 핵심 파일이나 다른 프로세스의 메모리에 접근할 수 없게 합니다. 이렇게 하면 악성 코드가 시스템을 손상시키는 것을 방지할 수 있습니다.
2. **안정성**
   * 사용자 애플리케이션의 오류가 전체 시스템에 영향을 미치지 않게 함
   * ex) 웹 브라우저에서 오류가 발생해 크래시가 발생한다 해도, 이 오류가 운영체제 전체에 영향을 미치지 않아 전체 시스템이 다운되지 않게 됩니다.
3. **추상화**
   * 하드웨어와 응용 프로그램 간의 명확한 경계 제공.
   * ex) 드라이버는 하드웨어를 직접 제어하는 작업을 처리하지만, 사용자 애플리케이션은 이 드라이버를 통해 추상화된 인터페이스로 하드웨어와 상호작용합니다.
