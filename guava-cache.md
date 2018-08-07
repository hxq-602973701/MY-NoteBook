-- 谷歌GUAVA 缓存
```java
    /**
     * 缓存
     */
    private static Cache<String, Object> cache = CacheBuilder.newBuilder().maximumSize(1000).expireAfterAccess(15, TimeUnit.DAYS).build();
    
      try {
            return (Map) cache.get("userMap", () -> {
                Map<Long, String> userMap = userService.select(new User()).parallelStream().collect(Collectors.toMap(User::getUserId, User::getUserName));
                cache.put("userMap", userMap);
                return userMap;
            });

        } catch (ExecutionException e1) {
            throw new DataErrorException(ResponseCode.SC_INTERNAL_SERVER_ERROR);
        }
