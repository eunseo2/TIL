# Cache
![image](https://github.com/eunseo2/TIL/assets/70589857/0324a481-da8d-4b19-b20c-ec93fe4558d3)

- 값비싼 연산 결과 또는 자주 참조되는 데이터를 메모리 안에 두고 사용하도록 하는 저장소입니다. 


## Cache 필요성 
![image](https://github.com/eunseo2/TIL/assets/70589857/af5e0591-7d8a-4820-810a-703b360c1e41)


위의 그래프는 Long Tail 법칙의 그래프입니다. Long Tail 법칙은 20%의 요구가 시스템 리소스의 대부분을 사용한다는 법칙입니다. 
그렇기 때문에 20%의 기능에 Cache를 이용함으로써 리소스 사용량울 대폭 줄이고, 성능은 대폭 향상시킬 수 있습니다.

------------------------------------------------------------------------------------------
### 1. Local Cache
![image](https://github.com/eunseo2/TIL/assets/70589857/1138b276-0c17-4f99-9394-c947b70eda52)


- 네트워크를 호출하지 않으며, 서버의 메모리에 직접 접근하기 때문에 빠릅니다.
- 서버가 여러대인 경우 동기화 문제가 발생할 수 있으므로, 단일 서버 혹은 변경이 드문 데이터에 대해서만 Local Cache가 적합합니다. 
 <br/><br/>
### 테스트   

- Asset 데이터는 변하지 않으며, 사용자에게 제공함으로 로컬 캐시에 적합한 데이터입니다.

  ![image](https://github.com/eunseo2/TIL/assets/70589857/b777c633-c49a-4943-b6b3-2a8392f3e429)


```
    //caffeine
    implementation 'org.springframework.boot:spring-boot-starter-cache:3.1.5'
    implementation 'com.github.ben-manes.caffeine:caffeine:3.1.8'
```

```java
@Configuration
@EnableCaching
public class CacheConfig {
	@Bean
	public CacheManager cacheManager() {
		CaffeineCacheManager cacheManager = new CaffeineCacheManager("Cache");
		cacheManager.setCaffeine(caffeineConfig());
		return cacheManager;
	}

	private Caffeine<Object, Object> caffeineConfig() {
		return Caffeine.newBuilder()
			.expireAfterWrite(1, TimeUnit.MINUTES)
			.maximumSize(100);
	}
}
```
- cache ttl 1분(MINUTES)으로 적용
- cache maximumSize 100


```java
	@Cacheable(value = "Cache", key = "#platform")
	public List<AssetResponseDto> getAssets(AssetPlatform platform) throws InterruptedException {
		Thread.sleep(10000);
		return assetRepository.getAssetResponseDto(platform, UseStatus.USE);
	}
```
> Local Cache 적용 전 

Thread.sleep(10000) 으로 인해 ${\small{\color{#DD6565}10.28s}}$ 가 걸리는 것을 볼 수 있습니다. 
![image](https://github.com/eunseo2/TIL/assets/70589857/f2a9caf8-375d-42ea-be2e-afd2447b17f3)

> Local Cache 적용 후

반면, 캐싱이 적용되어 ${\small{\color{#DD6565}59ms}}$ 가 걸립니다. 
![image](https://github.com/eunseo2/TIL/assets/70589857/191df7a6-5c08-4af9-8478-05efdf447647)


### 2. Global Cache
![image](https://github.com/eunseo2/TIL/assets/70589857/a17213b6-5ee5-4079-a030-e4e82a7a8b4b)


- Global Cache는 서버와 분리된 별도 저장소에 데이터를 보관합니다.
- 서버 간에 동기화가 필요하지 않습니다.
- 네트워크 호출이 필요하며, 상대적으로 로컬 캐시보다 느립니다.

### 테스트  
```java
@Slf4j
@Configuration
@EnableCaching
public class CacheConfig {
	public static final String LOCAL_CACHE_MANAGER = "localCacheManager";
	public static final String REDIS_CACHE_MANAGER = "redisCacheManager";

	@Bean(name = LOCAL_CACHE_MANAGER)
	@Primary
	public CacheManager localCacheManager() {
		log.info("local cache");
		CaffeineCacheManager cacheManager = new CaffeineCacheManager("Asset");
		cacheManager.setCaffeine(caffeineConfig());
		return cacheManager;
	}

	@Bean(name = REDIS_CACHE_MANAGER) // Redis cache manager 추가 
	public CacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
		log.info("redis cache");
		RedisCacheConfiguration cacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
			.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
			.serializeValuesWith(
				RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
			.entryTtl(Duration.ofMinutes(1)) //ttl 적용 
			.disableCachingNullValues();

		return RedisCacheManager.builder(redisConnectionFactory)
			.cacheDefaults(cacheConfiguration)
			.build();
	}

	private Caffeine<Object, Object> caffeineConfig() {
		return Caffeine.newBuilder()
			.expireAfterWrite(1, TimeUnit.MINUTES)
			.maximumSize(100);
	}
}

```

```java
	@Override
	@Cacheable(value = "Asset", key = "#platform", cacheManager = REDIS_CACHE_MANAGER) //cache manager 지정 
	public List<AssetResponseDto> getAssetsRedisCache(AssetPlatform platform) throws InterruptedException {
		Thread.sleep(3000);
		return assetRepository.getAssetResponseDto(platform, UseStatus.USE);
	}
```
<br></br>
> Global Cache 적용 전 
![image](https://github.com/eunseo2/TIL/assets/70589857/a3624a63-bccf-4187-82ce-54981cfc65a1)


Thread.sleep(3000)으로 인해  ${\small{\color{#DD6565}3.92s}}$ 가 걸리는 것을 볼 수 있습니다.

<br></br>
> Global Cache 적용 후 
![image](https://github.com/eunseo2/TIL/assets/70589857/4542e81b-667b-453e-a855-487f0aacefac)


캐싱이 적용되어 ${\small{\color{#DD6565}212ms}}$가 걸립니다.

Redis는 메모리 기반의 키-값 저장소로 Asset::WEB 키가 생성된 것을 볼 수 있습니다. 

![image](https://github.com/eunseo2/TIL/assets/70589857/27715c24-7de4-4e1d-8bcb-0a4a95928bc6)


### 참고자료
https://github.com/ben-manes/caffeine

https://mangkyu.tistory.com/69
