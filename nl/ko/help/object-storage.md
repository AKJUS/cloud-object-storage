---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-03-19"

keywords: about, basics

subcollection: cloud-object-storage

---
{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:note: .note}
{:download: .download} 
{:http: .ph data-hd-programlang='http'} 
{:javascript: .ph data-hd-programlang='javascript'} 
{:java: .ph data-hd-programlang='java'} 
{:python: .ph data-hd-programlang='python'}

# 오브젝트 스토리지 정보
{: #about-cos}

오브젝트 스토리지는 최신 스토리지 기술 개념이며 블록 및 파일 스토리지에서 논리적으로 발전된 형태입니다. 오브젝트 스토리지는 1990년대 후반부터 시작되었지만 지난 10년 동안 시장에 수용되어 성공을 거두었습니다.

오브젝트 스토리지는 여러 가지 문제를 해결하기 위해 개발되었습니다.

*  기존의 블록 및 파일 시스템을 사용한 대규모 데이터 관리는 데이터 관리 하드웨어 및 소프트웨어 스택의 다양한 레벨에 대한 한계로 인해 데이터 섬이 발생하기 때문에 어려웠습니다.

*  적정 규모로 네임스페이스를 관리하여 데이터 액세스에 필요한 크고 복잡한 계층 구조를 유지했습니다. 기존 블록 및 파일 스토리지 배열의 중첩된 구조에 대한 한계로 인해 데이터 섬이 형성되었습니다.

*  액세스 보안을 제공하기 위해 기술의 조합, 복잡한 보안 스킴 및 이러한 영역 관리에 대한 많은 인적 개입이 필요했습니다.

오브젝트 기반 스토리지(OBS)라고도 하는 오브젝트 스토리지는 다른 방식을 사용하여 데이터를 저장하고 참조합니다. 오브젝트 데이터 스토리지 개념에는 다음 세 가지 구성이 포함됩니다.

*  데이터: 이는 지속적 스토리지가 필요한 사용자 및 애플리케이션 데이터입니다. 텍스트, 2진 형식, 멀티미디어 또는 기타 사용자 생성 컨텐츠나 머신 생성 컨텐츠일 수 있습니다.

*  메타데이터: 이는 데이터에 대한 데이터입니다. 업로드 시간 및 크기와 같은 사전 정의된 몇 가지 속성이 포함됩니다. 오브젝트 스토리지를 사용하면 사용자가 키 및 값 쌍의 정보가 있는 사용자 정의 메타데이터를 포함할 수 있습니다. 이 정보에는 일반적으로 데이터를 저장하는 사용자나 애플리케이션 관련 정보가 들어 있으며, 이는 언제든 수정 가능합니다. 오브젝트 스토리지 시스템에서 처리하는 메타데이터의 특별한 점은 메타데이터가 오브젝트와 함께 저장된다는 것입니다.

*  키: 고유 리소스 ID가 OBS 시스템의 모든 오브젝트에 지정됩니다. 이 키는 오브젝트 스토리지 시스템이 오브젝트를 서로 구별할 수 있게 하며 정확한 실제 드라이브, 배열 또는 데이터가 있는 사이트를 알 필요 없이 데이터를 찾는 데 사용됩니다.

이 방식을 통해 오브젝트 스토리지는 단순한 일반 계층 구조에 데이터를 저장할 수 있으며, 이로 인해 성능이 저하되는 대용량의 메타데이터 저장소에 대한 필요성이 줄어듭니다.

데이터 액세스는 HTTP 프로토콜을 통해 REST 인터페이스를 사용하여 수행되며, 간단하게 오브젝트 키를 참조하여 언제 어디서나 액세스할 수 있습니다.