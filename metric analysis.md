1. 메트릭 시스템 아키텍처:

MinIO Metrics System
├── Metrics Collection Layer
│   ├── Resource Metrics (metrics-resource.go)
│   ├── Real-time Metrics (metrics-realtime.go)
│   └── Scanner Metrics (data-scanner-metric.go)
│
├── Metrics Processing Layer
│   ├── Metrics V2 (metrics-v2.go)
│   └── Metrics V3 (metrics-v3.go)
│
└── Metrics Export Layer
    ├── Prometheus Export
    └── Admin API Export


2. 주요 메트릭 카테고리:

const (
    // 시스템 메트릭
    systemNetworkInternodeCollectorPath = "/system/network/internode"
    systemDriveCollectorPath           = "/system/drive"
    systemMemoryCollectorPath          = "/system/memory"
    systemCPUCollectorPath             = "/system/cpu"
    systemProcessCollectorPath         = "/system/process"

    // 클러스터 메트릭
    clusterHealthCollectorPath       = "/cluster/health"
    clusterUsageObjectsCollectorPath = "/cluster/usage/objects"
    clusterUsageBucketsCollectorPath = "/cluster/usage/buckets"
    clusterErasureSetCollectorPath   = "/cluster/erasure-set"
    clusterIAMCollectorPath          = "/cluster/iam"
    clusterConfigCollectorPath       = "/cluster/config"

    // API 메트릭
    apiRequestsCollectorPath = "/api/requests"
    bucketAPICollectorPath   = "/bucket/api"

    // 기타 메트릭
    ilmCollectorPath           = "/ilm"
    auditCollectorPath         = "/audit"
    loggerWebhookCollectorPath = "/logger/webhook"
    replicationCollectorPath   = "/replication"
    notificationCollectorPath  = "/notification"
    scannerCollectorPath       = "/scanner"
)


3. 메트릭 수집 구조:

// 메트릭 그룹 구조
type MetricsGroup struct {
    CollectorPath collectorPath
    Descriptors   []MetricDescriptor
    Loader        func(ctx context.Context) []MetricV2
    Cache         *metricsCache
}

// 메트릭 데이터 구조
type MetricV2 struct {
    Description          MetricDescription
    Value               float64
    VariableLabels      map[string]string
    StaticLabels        map[string]string
    Histogram          map[string]uint64
    HistogramBucketLabel string
}

4. 주요 메트릭 수집기:
// 리소스 메트릭 수집기
type minioResourceCollector struct {
    metricsGroups []*MetricsGroupV2
    desc         *prometheus.Desc
}

// 클러스터 메트릭 수집기
type minioClusterCollector struct {
    metricsGroups []*MetricsGroupV2
    desc         *prometheus.Desc
}

// 버킷 메트릭 수집기
type minioBucketCollector struct {
    metricsGroups []*MetricsGroupV2
    desc         *prometheus.Desc
}

5. 메트릭 수집 프로세스:
1. 메트릭 수집 시작
   ├── 로컬 메트릭 수집 (collectLocalMetrics)
   │   ├── 디스크 메트릭
   │   ├── CPU 메트릭
   │   ├── 메모리 메트릭
   │   └── 네트워크 메트릭
   │
   └── 원격 메트릭 수집 (collectRemoteMetrics)
       └── 피어 노드로부터 메트릭 수집

2. 메트릭 처리
   ├── 메트릭 그룹화
   ├── 캐싱
   └── 집계

3. 메트릭 내보내기
   ├── Prometheus 형식으로 변환
   └── Admin API를 통한 노출

6. 메트릭 수집 프로세스:
// 리소스 메트릭
const (
    // 디스크 메트릭
    totalInodes    = "total_inodes"
    readsPerSec    = "reads_per_sec"
    writesPerSec   = "writes_per_sec"
    readsKBPerSec  = "reads_kb_per_sec"
    writesKBPerSec = "writes_kb_per_sec"
    
    // 네트워크 메트릭
    interfaceRxBytes  = "rx_bytes"
    interfaceTxBytes  = "tx_bytes"
    interfaceRxErrors = "rx_errors"
    interfaceTxErrors = "tx_errors"
    
    // CPU 메트릭
    cpuUser   = "user"
    cpuSystem = "system"
    cpuIOWait = "iowait"
    cpuIdle   = "idle"
)

7. 메트릭 캐싱 및 갱신:
// 메트릭 캐시 구조
type metricsCache struct {
    mu      sync.RWMutex
    metrics []MetricV2
    updated time.Time
}

// 캐시 갱신 주기
const (
    resourceMetricsCollectionInterval = time.Minute
    resourceMetricsCacheInterval     = time.Minute
)

이 메트릭 시스템은 다음과 같은 특징을 가집니다:
1. 계층적 구조: 시스템, 클러스터, API 등 다양한 계층의 메트릭을 수집
2. 실시간 모니터링: 리소스 사용량, 성능 지표 등을 실시간으로 수집
3. 확장성: 새로운 메트릭 타입을 쉽게 추가할 수 있는 구조
4. 분산 수집: 클러스터 환경에서 모든 노드의 메트릭을 수집
5. Prometheus 통합: Prometheus 형식으로 메트릭을 내보내 모니터링 시스템과 통합
6. 캐싱 메커니즘: 성능 최적화를 위한 메트릭 캐싱. 
이 메트릭 시스템을 통해 MinIO 클러스터의 상태, 성능, 리소스 사용량 등을 종합적으로 모니터링할 수 있습니다.

-----
Metric V2와 V3의 코드를 비교 분석해보겠습니다.

코드를 분석한 결과, Metric V2와 V3의 주요 공통점과 차이점을 다음과 같이 정리할 수 있습니다:
### 공통점
1. 기본 메트릭 구조
- 두 버전 모두 메트릭의 기본 구조를 정의하는 타입을 가지고 있습니다:
    - V2: MetricV2 구조체: V2 (cmd/metrics-v2.go)
    ```
    // MetricV2 구조체 정의
    type MetricV2 struct {
        Description          MetricDescription `json:"Description"`
        StaticLabels         map[string]string `json:"StaticLabels"`
        Value                float64           `json:"Value"`
        VariableLabels       map[string]string `json:"VariableLabels"`
        HistogramBucketLabel string            `json:"HistogramBucketLabel"`
        Histogram            map[string]uint64 `json:"Histogram"`
    }
    ```
    - V3: Metric 구조체: V3 (cmd/metrics-v3-types.go)
    ```
    // Metric 구조체 정의
    type Metric struct {
        Description MetricDescription
        Value       float64
        Labels      map[string]string
    }
    ```
- 두 버전 모두 메트릭 설명(Description), 값(Value), 레이블(Labels) 등의 기본 속성을 포함합니다.
2. 메트릭 그룹화
- 두 버전 모두 메트릭을 그룹화하는 개념을 사용합니다:
    - V2: MetricsGroupV2: 2 (cmd/metrics-v2.go)
    ```
    // MetricsGroupV2 구조체
    type MetricsGroupV2 struct {
        metricsCache     *cachevalue.Cache[[]MetricV2]
        cacheInterval    time.Duration
        metricsGroupOpts MetricsGroupOpts
    }
    ```
    - V3: MetricsGroup: V3 (cmd/metrics-v3-types.go)
    ```
    // MetricsGroup 구조체
    type MetricsGroup struct {
        CollectorPath collectorPath
        Descriptors   []MetricDescriptor
        ExtraLabels   map[string]string
        loader        MetricsLoaderFn
        bucketLoader  BucketMetricsLoaderFn
        cache         *metricsCache
        // ... 기타 필드들
    }
    ```
- 그룹화를 통해 관련된 메트릭들을 논리적으로 구성합니다.
3. 캐싱 메커니즘
- 두 버전 모두 메트릭 데이터의 캐싱을 지원합니다:
    - V2: metricsCache 필드를 통한 캐싱: V2 (cmd/metrics-v2.go)
    ```
    // 캐시 사용 예시
    func (g *MetricsGroupV2) Get() (metrics []MetricV2) {
        m, _ := g.metricsCache.Get()
        // ... 캐시 처리 로직
    }
    ```
    - V3: metricsCache 구조체를 통한 캐싱: V3 (cmd/metrics-v3-cache.go)
    ```
    // metricsCache 구조체
    type metricsCache struct {
        dataUsageInfo       *cachevalue.Cache[DataUsageInfo]
        esetHealthResult    *cachevalue.Cache[HealthResult]
        driveMetrics        *cachevalue.Cache[storageMetrics]
        memoryMetrics       *cachevalue.Cache[madmin.MemInfo]
        cpuMetrics          *cachevalue.Cache[madmin.CPUMetrics]
        clusterDriveMetrics *cachevalue.Cache[storageMetrics]
        nodesUpDown         *cachevalue.Cache[nodesOnline]
    }
    ```
4. 메트릭 타입
- 두 버전 모두 게이지(gauge), 카운터(counter) 등의 메트릭 타입을 지원합니다.
### 차이점
1. API 구조
- V2: V2 (cmd/metrics-router.go)
    - 레거시 엔드포인트 사용 (/prometheus/metrics)
    - 더 단순한 라우팅 구조
    ```
    // V2 엔드포인트 정의
    const (
        prometheusMetricsPathLegacy     = "/prometheus/metrics"
        prometheusMetricsV2ClusterPath  = "/v2/metrics/cluster"
        prometheusMetricsV2BucketPath   = "/v2/metrics/bucket"
        prometheusMetricsV2NodePath     = "/v2/metrics/node"
        prometheusMetricsV2ResourcePath = "/v2/metrics/resource"
    )
    ```
- V3:V3 (cmd/metrics-router.go)
    - 새로운 엔드포인트 사용 (/metrics/v3)
    - 더 체계적인 라우팅 구조
    - 버킷별 메트릭을 위한 전용 엔드포인트 지원
    ```
    // V3 엔드포인트 정의
    const (
        metricsV3Path = "/metrics/v3"
    )

    // V3 라우터 등록
    metricsRouter.Methods(http.MethodGet).Path(metricsV3Path + "{pathComps:.*}").Handler(metricsV3Server)
    ```
2. 메트릭 구성
- V2:V2 (cmd/metrics-v2.go)
    - 더 단순한 메트릭 구성
    - 정적 레이블과 동적 레이블의 구분이 덜 명확
    ```
    // V2 메트릭 그룹 옵션
    type MetricsGroupOpts struct {
        dependGlobalObjectAPI           bool
        dependGlobalAuthNPlugin         bool
        dependGlobalSiteReplicationSys  bool
        // ... 기타 옵션들
    }
    ```
- V3:V3 (cmd/metrics-v3-types.go)
    - 더 구조화된 메트릭 구성
    - 버킷 메트릭을 위한 전용 지원
    - 메트릭 설명자(Descriptor)의 명확한 분리
    ```
    // V3 메트릭 그룹 생성
    func NewMetricsGroup(path collectorPath, descriptors []MetricDescriptor,
        loader MetricsLoaderFn) *MetricsGroup {
        mg := &MetricsGroup{
            CollectorPath: path,
            Descriptors:   descriptors,
            loader:        loader,
        }
        mg.validate()
        return mg
    }

    // 버킷 메트릭을 위한 전용 그룹 생성
    func NewBucketMetricsGroup(path collectorPath, descriptors []MetricDescriptor,
        loader BucketMetricsLoaderFn) *MetricsGroup {
        // ... 버킷 메트릭 그룹 생성 로직
    }
    ```

3. 확장성
- V2:
    - 기본적인 확장성 지원
    - 메트릭 추가가 상대적으로 덜 체계적
- V3:
    - 더 나은 확장성 지원
    - 메트릭 그룹의 체계적인 관리
    - 버킷별 메트릭을 위한 전용 인터페이스
4. 성능 최적화
- V2:V2 (cmd/metrics-v2.go)
    - 기본적인 캐싱 메커니즘
    ```
    // V2 캐시 간격 설정
    func getNetworkMetrics() *MetricsGroupV2 {
        mg := &MetricsGroupV2{
            cacheInterval: 10 * time.Second, // 단순한 캐시 간격
        }
        // ... 구현
    }
    ```
- V3:V3 (cmd/metrics-v3-cache.go)
    - 더 정교한 캐싱 전략
    - 메트릭 로딩의 최적화
    - 버킷 메트릭을 위한 효율적인 데이터 구조
    ```
    // V3 캐시 구조
    type metricsCache struct {
        dataUsageInfo       *cachevalue.Cache[DataUsageInfo]
        esetHealthResult    *cachevalue.Cache[HealthResult]
        driveMetrics        *cachevalue.Cache[storageMetrics]
        // ... 다양한 캐시 타입
    }

    // 캐시 생성 함수
    func newMetricsCache() *metricsCache {
        return &metricsCache{
            dataUsageInfo:       newDataUsageInfoCache(),
            esetHealthResult:    newESetHealthResultCache(),
            driveMetrics:        newDriveMetricsCache(),
            // ... 다양한 캐시 초기화
        }
    }
    ```
5. 코드 구조
- V2:
    - 단일 파일에 많은 기능이 집중
    - 상대적으로 덜 모듈화된 구조
- V3:
    - 기능별로 분리된 파일 구조
    - 더 모듈화된 코드 구조
    - 명확한 책임 분리
6. 메트릭 수집 방식
- V2: V2 (cmd/metrics-v2.go)
    - 단순한 메트릭 수집 방식
    - 직접적인 메트릭 값 설정
    ```
    // V2 메트릭 수집 예시
    func getNetworkMetrics() *MetricsGroupV2 {
        mg := &MetricsGroupV2{
            cacheInterval: 10 * time.Second,
        }
        mg.RegisterRead(func(ctx context.Context) (metrics []MetricV2) {
            // 직접적인 메트릭 값 설정
            metrics = make([]MetricV2, 0, 10)
            connStats := globalConnStats.toServerConnStats()
            // ... 메트릭 수집 로직
            return
        })
        return mg
    }
    ```
- V3: V3 (cmd/metrics-v3-cluster-usage.go)
    - 더 체계적인 메트릭 수집
    - 로더 함수를 통한 유연한 메트릭 수집
    - 버킷별 메트릭 수집 지원
    ```
    // V3 메트릭 수집 예시
    func loadClusterUsageObjectMetrics(ctx context.Context, m MetricValues, c *metricsCache) error {
        dataUsageInfo, err := c.dataUsageInfo.Get()
        if err != nil {
            metricsLogIf(ctx, err)
            return nil
        }
        // ... 체계적인 메트릭 수집 로직
        m.Set(usageSinceLastUpdateSeconds, time.Since(dataUsageInfo.LastUpdate).Seconds())
        m.Set(usageTotalBytes, float64(clusterSize))
        // ... 추가 메트릭 설정
        return nil
    }
    ```
7. 에러 처리
- V2:V2 (cmd/metrics-v2.go)
    - 기본적인 에러 처리
    ```
    // V2 에러 처리 예시
    func (g *MetricsGroupV2) Get() (metrics []MetricV2) {
        m, _ := g.metricsCache.Get() // 단순한 에러 무시
        // ... 처리 로직
    }
    ```
- V3:V3 (cmd/metrics-v3-cluster-usage.go)
    - 더 체계적인 에러 처리
    - 컨텍스트 기반의 에러 처리
    - 메트릭 수집 실패에 대한 더 나은 처리. 
    ```
    // V3 에러 처리 예시
    func loadClusterUsageObjectMetrics(ctx context.Context, m MetricValues, c *metricsCache) error {
        dataUsageInfo, err := c.dataUsageInfo.Get()
        if err != nil {
            metricsLogIf(ctx, err) // 컨텍스트 기반 에러 로깅
            return nil
        }
        // ... 처리 로직
    }
    ```

이러한 차이점들은 V3가 V2보다 더 체계적이고 확장 가능한 메트릭 시스템을 제공한다는 것을 보여줍니다. V3는 특히 버킷별 메트릭, 성능 최적화, 그리고 코드 구조화 측면에서 개선되었습니다.


------
Diagrams

1. 전체 아키텍처 비교 다이어그램:
graph TB
    subgraph "Metric V2 Architecture"
        V2[V2 Entry Point<br>/prometheus/metrics] --> V2Router[V2 Router]
        V2Router --> V2Cluster[/v2/metrics/cluster]
        V2Router --> V2Bucket[/v2/metrics/bucket]
        V2Router --> V2Node[/v2/metrics/node]
        V2Router --> V2Resource[/v2/metrics/resource]
        
        V2Cluster --> V2MetricsGroup[MetricsGroupV2]
        V2Bucket --> V2MetricsGroup
        V2Node --> V2MetricsGroup
        V2Resource --> V2MetricsGroup
        
        V2MetricsGroup --> V2Cache[Simple Cache]
        V2MetricsGroup --> V2Metric[MetricV2]
    end

    subgraph "Metric V3 Architecture"
        V3[V3 Entry Point<br>/metrics/v3] --> V3Router[V3 Router]
        V3Router --> V3Path[/metrics/v3/*]
        
        V3Path --> V3MetricsGroup[MetricsGroup]
        V3Path --> V3BucketMetricsGroup[BucketMetricsGroup]
        
        V3MetricsGroup --> V3Cache[Advanced Cache]
        V3BucketMetricsGroup --> V3Cache
        
        V3Cache --> V3DataUsage[DataUsage Cache]
        V3Cache --> V3Health[Health Cache]
        V3Cache --> V3Drive[Drive Cache]
        V3Cache --> V3Memory[Memory Cache]
        
        V3MetricsGroup --> V3Metric[Metric]
        V3BucketMetricsGroup --> V3Metric
    end

2. 메트릭 구조 비교 다이어그램:
classDiagram
    class MetricV2 {
        +MetricDescription Description
        +map[string]string StaticLabels
        +float64 Value
        +map[string]string VariableLabels
        +string HistogramBucketLabel
        +map[string]uint64 Histogram
    }
    
    class MetricV3 {
        +MetricDescription Description
        +float64 Value
        +map[string]string Labels
    }
    
    class MetricsGroupV2 {
        +*cachevalue.Cache[[]MetricV2] metricsCache
        +time.Duration cacheInterval
        +MetricsGroupOpts metricsGroupOpts
        +Get() []MetricV2
    }
    
    class MetricsGroupV3 {
        +collectorPath CollectorPath
        +[]MetricDescriptor Descriptors
        +map[string]string ExtraLabels
        +MetricsLoaderFn loader
        +BucketMetricsLoaderFn bucketLoader
        +*metricsCache cache
        +validate()
    }
    
    MetricsGroupV2 --> MetricV2
    MetricsGroupV3 --> MetricV3

3. 메트릭 수집 프로세스 비교 다이어그램:
sequenceDiagram
    participant Client
    participant V2Router
    participant V2Collector
    participant V2Cache
    participant V3Router
    participant V3Collector
    participant V3Cache
    
    %% V2 Flow
    Client->>V2Router: Request /prometheus/metrics
    V2Router->>V2Collector: Get Metrics
    V2Collector->>V2Cache: Check Cache
    V2Cache-->>V2Collector: Return Cached/New Data
    V2Collector-->>V2Router: Return Metrics
    V2Router-->>Client: Response
    
    %% V3 Flow
    Client->>V3Router: Request /metrics/v3
    V3Router->>V3Collector: Get Metrics
    V3Collector->>V3Cache: Check Multiple Caches
    V3Cache-->>V3Collector: Return Cached/New Data
    V3Collector->>V3Collector: Process Bucket Metrics
    V3Collector-->>V3Router: Return Metrics
    V3Router-->>Client: Response

4. 캐시 구조 비교 다이어그램:
graph TB
    subgraph "V2 Cache Structure"
        V2Cache[Simple Cache] --> V2MetricCache[Metric Cache]
    end
    
    subgraph "V3 Cache Structure"
        V3Cache[Advanced Cache] --> V3DataUsage[DataUsage Cache]
        V3Cache --> V3Health[Health Cache]
        V3Cache --> V3Drive[Drive Cache]
        V3Cache --> V3Memory[Memory Cache]
        V3Cache --> V3CPU[CPU Cache]
        V3Cache --> V3Cluster[Cluster Drive Cache]
        V3Cache --> V3Nodes[Nodes Up/Down Cache]
    end

5. 에러 처리 비교 다이어그램:
graph TB
    subgraph "V2 Error Handling"
        V2Error[Error Occurs] --> V2Ignore[Ignore Error]
        V2Error --> V2SimpleLog[Simple Logging]
    end
    
    subgraph "V3 Error Handling"
        V3Error[Error Occurs] --> V3ContextLog[Context-based Logging]
        V3Error --> V3MetricsLog[Metrics-specific Logging]
        V3Error --> V3ErrorReturn[Error Return]
        V3Error --> V3Recovery[Recovery Mechanism]
    end


이러한 다이어그램들은 Metric V2와 V3의 주요 차이점을 시각적으로 보여줍니다:
1. 아키텍처 비교: V3가 더 체계적이고 확장 가능한 구조를 가짐
2. 메트릭 구조: V3가 더 단순화되고 명확한 구조를 가짐
3. 수집 프로세스: V3가 더 복잡하지만 체계적인 프로세스를 가짐
4. 캐시 구조: V3가 더 세분화되고 효율적인 캐시 시스템을 가짐
5. 에러 처리: V3가 더 견고하고 체계적인 에러 처리 메커니즘을 가짐
이 다이어그램들은 코드의 실제 구현을 기반으로 하며, 각 버전의 특징과 개선 사항을 명확하게 보여줍니다.