# Replication 적용
![image](https://github.com/eunseo2/TIL/assets/70589857/f89145d7-d151-47bf-942a-149741e70c8e)


주 데이터베이스 (SOURCE_SERVER)
- 쓰기 연산 (INSERT, UPDATE, DELETE) 

부 데이터베이스 (REPLICA_SERVER)
- 읽기 연산 (SELECT) 
  

## 구현 목적
1. 스케일 아웃:
- 수평적으로 서버를 증설하여 트래픽을 대웅하는데 유연한 구조를 가질 수 있습니다.
  
![image](https://github.com/eunseo2/TIL/assets/70589857/84993438-820f-4c6f-b6c6-6cbfc0628534)



2. 데이터 백업:
- 운영중인 서비스의 데이터 백업은 필수입니다.
- 백업 과정은 실제 실행중인 쿼리에 영향을 줄 수 있지만, 레플리카 서버에서 데이터 백업을 실행할 경우 소스 서버에서 백업시 발생하는 문제들을 해결할 수 있습니다.
 

3. 데이터 분석:
- 분석용 쿼리는 대량의 데이터를 조회하는 경우가 많습니다. 이러한 쿼리는 집계 연산 및 다른 무거운 연산을 수반하기 때문에 소스 서버보다는 레플리카 서버에서 데이터를 조회하는 것이 효율적입니다. 

4. 데이터의 지리적 분산
- 서비스에서 사용되는 에플리케이션 서버와 DB는 지리적으로 근접하거나 멀 수 있습니다. DB 서버와 에플리케이션 서버의 거리에 따라 통신 시간이 달라질 수 있으므로 다양한 지점에 대한 DB 서버를 위치함으로써 응답 속도를 개선시킬 수 있습니다.


## Replication 작동 원리
![image](https://github.com/eunseo2/TIL/assets/70589857/7c61c9fa-f73e-4dd2-84fd-044bdd4ed311)


- 소스서버의 변경을 기록하기 위한 바이너리 로그
- 레플리카 서버에 데이터를 전송하기 위한 마스터 스레드
- 레플리카 서버에서 데이터를 받아 릴레이 로그에 기록하기 위한 I/O 스레드
- 릴레이 로그에서 데이터를 읽어 재생하기 위한 레플리카 SQL 스레드

## Spring 적용
```yml
  datasource:
    source:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbc-url: jdbc:mysql://localhost:3306/source?serverTimezone=UTC&characterEncoding=UTF-8
      username: root
      password: 1234
    replica:
      driver-class-name: com.mysql.cj.jdbc.Driver
      jdbc-url: jdbc:mysql://localhost:3306/replica?serverTimezone=UTC&characterEncoding=UTF-8
      username: root
      password: 1234
```
```java
@Slf4j
@Configuration
public class DataSourceConfig {
	@Bean
	@ConfigurationProperties("spring.datasource.source")
	public DataSource sourceDataSource() {
		log.info("source register");
		return DataSourceBuilder.create().build();
	}

	@Bean
	@ConfigurationProperties("spring.datasource.replica")
	public DataSource replicaDataSource() {
		log.info("replica register");
		return DataSourceBuilder.create().build();
	}

	@Bean
	public DataSource routingDataSource(
		DataSource sourceDataSource,
		DataSource replicaDataSource
	) {
		RoutingDataSource routingDataSource = new RoutingDataSource();

		HashMap<Object, Object> dataSourceMap = new HashMap<>();
		dataSourceMap.put(DataSourceType.SOURCE.name(), sourceDataSource);
		dataSourceMap.put(DataSourceType.REPLICA.name(), replicaDataSource);

		routingDataSource.setTargetDataSources(dataSourceMap);
		routingDataSource.setDefaultTargetDataSource(sourceDataSource);

		return routingDataSource;
	}

	@Bean
	@Primary
	public DataSource dataSource() {
		DataSource determinedDataSource = routingDataSource(sourceDataSource(), replicaDataSource());
		return new LazyConnectionDataSourceProxy(determinedDataSource);
	}

	public enum DataSourceType {
		SOURCE,
		REPLICA
	}

	public class RoutingDataSource extends AbstractRoutingDataSource {
		@Nullable
		@Override
		protected Object determineCurrentLookupKey() {
			String datasource = TransactionSynchronizationManager.isCurrentTransactionReadOnly()
				? DataSourceType.REPLICA.name()
				: DataSourceType.SOURCE.name();
			log.info("Current DataSource is {}", datasource);
			return datasource;
		}
	}
}


```
@Transactional(readOnly = true) 일 경우 DataSourceType{REPLICA}이며, 기본값은 DataSourceType{SOURCE} 이다. 


### 참고자료
https://mariadb.com/kb/en/setting-up-replication

https://cheese10yun.github.io/spring-transaction

https://www.youtube.com/watch?v=95bnLnIxyWI
