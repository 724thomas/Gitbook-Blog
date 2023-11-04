---
description: IPC Shared Memory기법의 프로세스 주소 공간
---

# Process address space in IPC Shared Memory technique

**위치**

보통 힙 영역과 스택 영역 사이의 공간인 MMS(Memory Mapping Segment)에 위치할 수 있습니다. 다른 위치에 매핑될 수도 있으며, 이는 운영체제와 구현에 따라 다릅니다.

**이유**

**중앙집중화** - MMS는 공유 라이브러리, 메모리 매핑 파일, 공유 메모리 영역과 같은 여러 프로세스 간에 공유되는 리소스에 대한 중앙화된 위치를 제공합니다. 이러한 리소스를 중앙에 위치시키면 공유 엔터티의 관리가 단순화됩니다.
