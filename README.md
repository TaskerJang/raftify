# 🚀 Raftify: Rust 기반 분산 합의 시스템

## 🎯 소개
2024 오픈소스 컨트리뷰션 아카데미(OSSCA) 프로젝트로, Raft 합의 알고리즘의 Rust 구현체에 RocksDB 스토리지를 통합했습니다.

### ⏰ 프로젝트 기간
- **전체**: 2024.7.13 - 2024.11.2 (13주)
- **Challenges**: 2024.7.13 - 2024.8.9
- **Masters**: 2024.8.10 - 2024.11.2

## 💻 기술 스택
- **언어**: Rust
- **스토리지**: RocksDB
- **통신**: gRPC
- **웹 프레임워크**: Actix-web
- **직렬화**: Protobuf, Serde

## 🔧 구현 내용

### 1. Raft 알고리즘 구현
#### 핵심 컴포넌트
- **리더 선출**: 랜덤화된 타이머 기반 선출 시스템
- **로그 복제**: 리더 주도 로그 동기화
- **상태 관리**: Term 기반 상태 추적

#### 주요 특징
```rust
pub trait StableStorage: Storage {
    const STORAGE_TYPE: StorageType;
    fn append(&mut self, entries: &[Entry]) -> Result<()>;
    fn hard_state(&self) -> Result<HardState>;
    // ... 기타 메서드
}
```

### 2. RocksDB 스토리지 구현

#### 핵심 기능
```rust
pub struct RocksDBStorage {
    db: DB,
    logger: Arc<dyn Logger>,
}

impl RocksDBStorage {
    pub fn create(path: &str, logger: Arc<dyn Logger>) -> Result<Self> {
        let mut opts = Options::default();
        opts.create_if_missing(true);
        let db = DB::open(&opts, path)?;
        Ok(RocksDBStorage { db, logger })
    }
}
```

#### 주요 컴포넌트
1. **Write Ahead Logging (WAL)**
   - 데이터 영속성 보장
   - 장애 복구 메커니즘

2. **MemTable**
   - 인메모리 데이터 구조
   - 빠른 읽기/쓰기 지원

3. **SST (Sorted String Table) Files**
   - 계층화된 데이터 저장
   - 효율적인 압축

4. **Compaction**
   - 자동 로그 압축
   - 스토리지 최적화

5. **Bloom Filter**
   - 효율적인 키 검색
   - 불필요한 디스크 접근 방지

### 3. HTTP API 개선
```rust
#[post("/leave")]  // 상태 변경 작업은 POST로 수정
async fn leave(data: web::Data<(HashStore, Raft)>) -> impl Responder {
    let raft = data.clone();
    raft.1.leave().await.unwrap();
    "OK".to_string()
}

#[get("/status")]  // 상태 조회는 GET 유지
async fn status(data: web::Data<(HashStore, Raft)>) -> impl Responder {
    // ... 상태 조회 로직
}
```

## 📦 설치 및 실행

### 의존성 설정
```toml
[dependencies]
raftify = { version = "0.1.78", features = ["rocksdb_storage"] }
rocksdb = "0.19.0"
```

### 스토리지 설정
```toml
[features]
default = ["heed_storage"]
rocksdb_storage = ["rocksdb"]
inmemory_storage = []
heed_storage = ["heed", "heed-traits"]
```

## 🏗 아키텍처

### 시스템 구조
```
src/
├── storage/
│   ├── rocksdb_storage/    # RocksDB 구현
│   ├── heed_storage/       # Heed 구현
│   └── inmemory_storage/   # 인메모리 구현
├── raft/                   # Raft 알고리즘 코어
└── api/                    # HTTP API 레이어
```

## 📚 기술 문서

### RocksDB 관련
- [Write Ahead Logging](https://velog.io/@tasker_dev/Write-Ahead-Logging)
- [MemTable 구조](https://velog.io/@tasker_dev/MemTable)
- [SST Files](https://velog.io/@tasker_dev/SST-Files)
- [Compaction](https://velog.io/@tasker_dev/Compaction)
- [Bloom Filter](https://velog.io/@tasker_dev/Bloom-Filter)
- [Transaction Log](https://velog.io/@tasker_dev/11.-Transaction-Log)

### Raft 알고리즘
- [Raft 배경 지식](https://velog.io/@tasker_dev/1.-러스트에-들어가기-앞서-배경지식)
- [합의 알고리즘 이해](https://velog.io/@tasker_dev/In-Search-of-an-Understandable-Consensus-Algorithm)
- [코드 분석](https://velog.io/@tasker_dev/Raft-코드-뜯어보기)

## 🎉 프로젝트 성과

### Pull Requests
1. [#124 RocksDB 스토리지 구현](https://github.com/lablup/raftify/pull/124)
   - RocksDB 백엔드 통합
   - 성능 최적화
   - 테스트 케이스 구현

2. [#147 HTTP API 개선](https://github.com/lablup/raftify/pull/147)
   - RESTful 원칙 준수
   - API 일관성 향상

## 🔜 향후 계획
- Java 기반 프로젝트 참여
- Raftify 지속적 개선
- 성능 최적화 작업

## 📝 참고 자료
- [RocksDB 공식 문서](https://rocksdb.org)
- [Raft 논문](https://raft.github.io)
- [프로젝트 블로그](https://velog.io/@tasker_dev)
