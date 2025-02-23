# 🚀 Raftify: Rust 기반 분산 합의 시스템

## 🎯 소개
2024 오픈소스 컨트리뷰션 아카데미(OSSCA) 프로젝트로, Raft 합의 알고리즘의 Rust 구현체에 RocksDB 스토리지를 통합했습니다.

### ⏰ 프로젝트 기간
- **전체**: 2024.7.13 - 2024.11.2 (13주)
- **Challenges**: 2024.7.13 - 2024.8.9
- **Masters**: 2024.8.10 - 2024.11.2

## 💻 기술 스택
![Rust](https://img.shields.io/badge/Rust-000000?style=for-the-badge&logo=rust&logoColor=white)
![Raftify](https://img.shields.io/badge/Raftify-E6484F?style=for-the-badge&logo=rust&logoColor=white)
![RocksDB](https://img.shields.io/badge/RocksDB-2C2C2C?style=for-the-badge&logo=rocksdb&logoColor=white)
![gRPC](https://img.shields.io/badge/gRPC-244c5a?style=for-the-badge&logo=grpc&logoColor=white)
![Actix](https://img.shields.io/badge/Actix-117777?style=for-the-badge&logo=rust&logoColor=white)
![Protobuf](https://img.shields.io/badge/Protobuf-4285F4?style=for-the-badge&logo=google&logoColor=white)
![Serde](https://img.shields.io/badge/Serde-000000?style=for-the-badge&logo=rust&logoColor=white)
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

<details>
<summary><b>Raft 알고리즘 시리즈</b></summary>

1. [Raft 알고리즘 배경 지식과 핵심 개념](https://velog.io/@tasker_dev/1.-러스트에-들어가기-앞서-배경지식)
2. [In Search of an Understandable Consensus Algorithm](https://velog.io/@tasker_dev/In-Search-of-an-Understandable-Consensus-Algorithm)
3. [Raft 코드 분석 1편](https://velog.io/@tasker_dev/Raft-코드-뜯어보기)
4. [Raft 코드 분석 2편](https://velog.io/@tasker_dev/Raft-코드-뜯어보기-2)
5. [Raft 코드 분석 3편](https://velog.io/@tasker_dev/Raft-코드-뜯어보기-3)
6. [Raft 코드 분석 4편](https://velog.io/@tasker_dev/Raft-코드-뜯어보기-4)

</details>

<details>
<summary><b>RocksDB 학습 시리즈</b></summary>

1. [RocksDB 소개](https://velog.io/@tasker_dev/Rocks-DB에-대해-알아보자)
2. [스토리지 구현 1편](https://velog.io/@tasker_dev/Rocks-DB-스토리지-구현-1)
3. [스토리지 구현 2편](https://velog.io/@tasker_dev/Rocks-DB-스토리지-구현-2-rh24xx3m)
4. [스토리지 구현 4편](https://velog.io/@tasker_dev/Rocks-DB-스토리지-구현-4)
5. [스토리지 구현 5편](https://velog.io/@tasker_dev/Rocks-DB-스토리지-구현-5)

</details>

<details>
<summary><b>RocksDB 핵심 개념</b></summary>

1. [Write Ahead Logging](https://velog.io/@tasker_dev/Write-Ahead-Logging)
2. [MemTable](https://velog.io/@tasker_dev/MemTable)
3. [SST Files](https://velog.io/@tasker_dev/SST-Files)
4. [Compaction](https://velog.io/@tasker_dev/Compaction)
5. [Bloom Filter](https://velog.io/@tasker_dev/Bloom-Filter)
6. [Transaction Log](https://velog.io/@tasker_dev/11.-Transaction-Log)
7. [Block Cache](https://velog.io/@tasker_dev/Block-Cache)
8. [Merge Operators](https://velog.io/@tasker_dev/9.-Merge-Operators)
9. [Iterators](https://velog.io/@tasker_dev/10.-Iterators)
10. [Statistics](https://velog.io/@tasker_dev/12.-statistics)
11. [Adaptive Mutex](https://velog.io/@tasker_dev/13.-Adaptive-Mutex)

</details>

<details>
<summary><b>API 개선</b></summary>

- [Raft 기반 시스템의 HTTP 메서드 선택](https://velog.io/@tasker_dev/Raft-기반-시스템에서-올바른-HTTP-메서드-선택과-API-개선-방법)

</details>

## 🎉 프로젝트 성과

<div align="center">
  
### 🚀 Pull Requests

</div>

<table>
<tr>
<td width="100%">
<div align="center">
  
[![PR](https://img.shields.io/badge/PR-124-brightgreen?style=for-the-badge&logo=github)](https://github.com/lablup/raftify/pull/124)

</div>

### 📦 RocksDB 스토리지 구현

> RocksDB를 Raftify의 백엔드 스토리지로 통합하고 성능을 최적화했습니다.

**주요 구현 사항**
- ✨ RocksDB 백엔드 통합
- 🚀 성능 최적화
- ✅ 테스트 케이스 구현
</td>
</tr>
<tr>
<td width="100%">
<div align="center">
  
[![PR](https://img.shields.io/badge/PR-147-brightgreen?style=for-the-badge&logo=github)](https://github.com/lablup/raftify/pull/147)

</div>

### 🔄 HTTP API 개선

> RESTful 원칙을 준수하여 API의 일관성과 사용성을 향상시켰습니다.

**주요 개선 사항**
- 📝 RESTful 원칙 준수
- ⚡ API 일관성 향상
</td>
</tr>
</table>

## 📖 전체 활동 후기

<div align="center">
  
[![Velog](https://img.shields.io/badge/Velog-2024_오픈소스_컨트리뷰션-20C997?style=for-the-badge&logo=velog&logoColor=white)](https://velog.io/@tasker_dev/2024-오픈소스-컨트리뷰션)

**13주간의 열정적인 오픈소스 여정을 기술 블로그에서 확인해보세요!**

</div>

## 🔗 참고 자료

<table>
<tr>
<td><a href="https://raft.github.io/">📑 Raft 논문</a></td>
<td><a href="https://rocksdb.org/">📚 RocksDB 공식 문서</a></td>
<td><a href="https://www.rust-lang.org/">🦀 Rust 공식 문서</a></td>
</tr>
</table>













