## 一、枚举

### 1.1 枚举【enum】

```java
@Getter
public enum InitMonth {

    JAN(1, "JAN"),
    FEB(2, "FEB"),
    MAR(3, "MAR"),
    APR(4, "APR"),
    MAY(5, "MAY");

    private int key;
    private String value;

    InitMonth(int key, String value) {
        this.key = key;
        this.value = value;
    }

    public static String of(int key) {
        for (InitMonth initMonth : values()) {
            if (initMonth.key == key) {
                return initMonth.getValue();
            }
        }
        throw new BusinessException("key值无法找到value");
    }
```



## 二、XML

### 2.1 foreach

```xml
 groupId in
<foreach collection="list" index="index" item="item" separator="," open="(" close=")">
    #{item}
</foreach>
```

### 2.2 where if

```xml
<where>
	 <if test="year != null and year != ''">
        and year = #{year}
    </if>
</where>
```



## 三、反射

### 3.1 反射调用方法

**动态方法反射调用：**

```java
Field[] declaredFields = drinkingWaterStatInfo.getClass().getDeclaredFields();
for (int i = 0; i < declaredFields.length; i++) {
    String name = declaredFields[i].getName();
    if (monthVal.equalsIgnoreCase(name)) {
        name = name.substring(0, 1).toUpperCase() + name.substring(1);
        try {
            Method method = drinkingWaterStatInfo.getClass().getMethod("get" + name);
            Integer invoke = (Integer) method.invoke(drinkingWaterStatInfo);
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```

## 四、文件流

### 4.1 大量数据缓存到临时文件

```java
 BufferedOutputStream bos = null;
 String localFileName = null;
try {
	 localFileName = System.getProperty("java.io.tmpdir") + File.separator + s + ".csv";
    bos = new BufferedOutputStream(new FileOutputStream(localFileName));
    boolean isEmpty = true;
    for (int i = 0; i < num + 1; i++) {
        List<ExternalTag> externalTagList1 = externalTagMapper.listInfo(k.toString(), tagName, 3000 * i);
        if (!CollectionUtils.isEmpty(externalTagList1)) {
            isEmpty = false;
            List<ExternalTagPo> externalTagPoList = new ArrayList<>();
            externalTagList1.forEach(a -> {
                ExternalTagPo externalTagPo = new ExternalTagPo();
                externalTagPo.setUnifyId(a.getUnifyId());
                externalTagPo.setTagValue(a.getTagValue());
                externalTagPoList.add(externalTagPo);
            });
            String[] headers = new String[]{"tag", "tagType", "valueType", "multiValue", "group"}; // excel表格头
            byte[] bytes = ExportCSVUtil.writeCsvAfterToBytes(headers, externalTagPoList, notHasValue);
            bos.write(bytes);
            bos.flush();
        }

    }
    if (!isEmpty) {
        InputStream input = new FileInputStream(localFileName);
        ossClient.putObject(bucket, uploadFileName, input);
        input.close();
    }

} catch (Exception e) {
    e.printStackTrace();
} finally {
    count.countDown();
    if (bos != null) {
        try {
            bos.close();
            //删除临时文件
            new File(localFileName).delete();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 4.2 每行读取文件

```java
InputStreamReader isr = new InputStreamReader(inputStream);
BufferedReader br = new BufferedReader(isr);
String lineTxt = br.readLine(); // 读取文件的方法【第一行表头给去掉了】
while ((lineTxt = br.readLine()) != null) {
    String[] arrStrings = lineTxt.split("\\|", -1); // 需要转义字符，并且多个要用-1表示
}
```



## 五、工具类

### 5.1 redisUtil

```java
public class RedisUtil {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;
	/**
     * 指定缓存失效时间
     * @param key 键
     * @param time 时间(秒)
     * @return 操作结果
     */
    public boolean expire(String key,long time){
        try {
            if(time>0){
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key){
        return redisTemplate.getExpire(key,TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key){
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除缓存
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String ... key){
        if(key!=null&&key.length>0){
            if(key.length==1){
                redisTemplate.delete(key[0]);
            }else{
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }

    //============================String=============================
    /**
     * 普通缓存获取
     * @param key 键
     * @return 值
     */
    public String get(String key){
        return key==null?null:redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     * @param key 键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key,String value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 普通缓存放入并设置时间
     * @param key 键
     * @param value 值
     * @param duration 时间 time要大于0 如果time等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key,String value,Duration duration){
        try {
            if(duration.getSeconds()>0){
                redisTemplate.opsForValue().set(key, value, duration.getSeconds(), TimeUnit.SECONDS);
            }else{
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 递增
     * @param key 键
     * @param delta 要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta){
        if(delta<0){
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     * @param key 键
     * @param delta 要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta){
        if(delta<0){
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    //================================Map=================================
    /**
     * HashGet
     * @param key 键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key,String item){
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object,Object> hmget(String key){
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String,String> map){
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     * @param key 键
     * @param map 对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String,String> map, long time){
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if(time>0){
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     * @param key 键
     * @param item 项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key,String item,String value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     * @param key 键
     * @param item 项
     * @param value 值
     * @param time 时间(秒)  注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key,String item,String value,long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if(time>0){
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除hash表中的值
     * @param key 键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item){
        redisTemplate.opsForHash().delete(key,item);
    }

    /**
     * 判断hash表中是否有该项的值
     * @param key 键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item){
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     * @param key 键
     * @param item 项
     * @param by 要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item,double by){
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     * @param key 键
     * @param item 项
     * @param by 要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item,double by){
        return redisTemplate.opsForHash().increment(key, item,-by);
    }

    //============================set=============================
    /**
     * 根据key获取Set中的所有值
     * @param key 键
     * @return
     */
    public Set<String> sGet(String key){
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     * @param key 键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key,String value){
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     * @param key 键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, String... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     * @param key 键
     * @param time 时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key,long time,String...values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if(time>0) {
                expire(key, time);
            }
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key){
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 移除值为value的
     * @param key 键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object ...values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    //===============================list=================================

    /**
     * 获取list缓存的内容
     * @param key 键
     * @param start 开始
     * @param end 结束  0 到 -1代表所有值
     * @return
     */
    public List<String> lGet(String key, long start, long end){
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     * @param key 键
     * @return
     */
    public long lGetListSize(String key){
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     * @param key 键
     * @param index 索引  index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public String lGetIndex(String key,long index){
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, String value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @param duration 时间
     * @return
     */
    public boolean lSet(String key, String value, Duration duration) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (duration.getSeconds() > 0) {
                expire(key, duration.getSeconds());
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<String> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @param time 时间(秒)
     * @return
     */
    public boolean lSet(String key, List<String> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     * @param key 键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index,String value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 移除N个值为value
     * @param key 键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key,long count,String value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 模糊查询获取key值
     * @param pattern
     * @return
     */
    public Set keys(String pattern){
        return redisTemplate.keys(pattern);
    }

    /**
     * 使用Redis的消息队列
     * @param channel
     * @param message 消息内容
     */
    public void convertAndSend(String channel, String message){
        redisTemplate.convertAndSend(channel,message);
    }


    //=========BoundListOperations 用法 start============

    /**
     *将数据添加到Redis的list中（从右边添加）
     * @param listKey listKey
     * @param duration 有效期
     * @param values 待添加的数据
     */
    public void addToListRight(String listKey, Duration duration, String... values) {
        //绑定操作
        BoundListOperations<String, String> boundValueOperations = redisTemplate.boundListOps(listKey);
        //插入数据
        boundValueOperations.rightPushAll(values);
        //设置过期时间
        boundValueOperations.expire(duration.getSeconds(),TimeUnit.SECONDS);
    }
    /**
     * 根据起始结束序号遍历Redis中的list
     * @param listKey listKey
     * @param start  起始序号
     * @param end  结束序号
     * @return list
     */
    public List<String> rangeList(String listKey, long start, long end) {
        //绑定操作
        BoundListOperations<String, String> boundValueOperations = redisTemplate.boundListOps(listKey);
        //查询数据
        return boundValueOperations.range(start, end);
    }
    /**
     * 弹出右边的值 --- 并且移除这个值
     * @param listKey listKey
     */
    public String rifhtPop(String listKey){
        //绑定操作
        BoundListOperations<String, String> boundValueOperations = redisTemplate.boundListOps(listKey);
        return boundValueOperations.rightPop();
    }

    //=========BoundListOperations 用法 End============

}
```

### 5.2 redis 分布式锁

```java
public class DistributedLock {

    @Autowired
    private RedisTemplate<String,String> redisTemplate;


    /**
     * 获取分布式锁
     * @param lockId 锁id
     * @param uuid 锁标记
     * @param timeout 锁自动超时时间
     * @return boolean
     */
    public Boolean lock(String lockId, String uuid, Duration timeout) {
        return redisTemplate.opsForValue().setIfAbsent(lockId,uuid,timeout);
    }

    public Boolean hasKey(String lockId) {
        return redisTemplate.hasKey(lockId);
    }

    /**
     * 等待获取分布式锁
     * @param lockId 锁id
     * @param uuid 锁标记
     * @param acquireTimeout 获取锁等待时间
     * @param timeout 锁自动超时时间
     * @return boolean
     */
    public Boolean lockWithTimeout(String lockId, String uuid, Duration acquireTimeout, Duration timeout) {
        long end = System.currentTimeMillis() + acquireTimeout.getSeconds() * 1000;
        while (System.currentTimeMillis() < end) {
            if (lock(lockId, uuid, timeout)) {
                return true;
            }
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        return false;
    }

    /**
     * 释放锁
     * @param lockId 锁id
     * @param uuid 锁标记
     */
    public void unlock(String lockId, String uuid) {
        String releaseLua = "if redis.call('get',KEYS[1]) == ARGV[1] then\n" +
                "return tostring(redis.call('del',KEYS[1]))\n" +
                "else\n" +
                "return '0'\n" +
                "end";
        RedisScript<Integer> releaseScript = new DefaultRedisScript<>(releaseLua,Integer.class);
        redisTemplate.execute(releaseScript, ImmutableList.of(lockId),uuid);
    }
}
```

>代码调用实例：

```java
String uuid = UUID.randomUUID().toString();
try {
    if(distributedLock.lock("cartier_cdp_campaignSync",uuid, Duration.ofSeconds(2))) {
        campaignSyncService.groupSync(null, null);
    }
} catch (Exception e) {
    log.error(e.getMessage(), e);
} finally {
    distributedLock.unlock("cartier_cdp_campaignSync", uuid);
}
```

### 5.3 CSV转换为字节流【byte】

```java
/**
 * 写CSV并转换为字节流
 * @param headers 表头
 * @param cellList 表数据
 * @return
 */
public static byte[] writeCsvAfterToBytes1(String[] headers,List<CustomerExternalTagPo> cellList) {
    byte[] bytes = new byte[0];
    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
    OutputStreamWriter outputStreamWriter = new OutputStreamWriter(byteArrayOutputStream, StandardCharsets.UTF_8);
    BufferedWriter bufferedWriter = new BufferedWriter(outputStreamWriter);
    CSVPrinter  csvPrinter = null;
    try {
        //创建csvPrinter并设置表格头
        csvPrinter = new CSVPrinter(bufferedWriter, CSVFormat.DEFAULT.withHeader(headers));
        //写数据
        for (CustomerExternalTagPo lineData : cellList) {
            csvPrinter.printRecord(new Object[]{lineData.getTagName(),lineData.getTagType(),lineData.getValueType(),lineData.getMultiValue(),lineData.getGroup()});
        }
        csvPrinter.flush();
        bytes = byteArrayOutputStream.toString(StandardCharsets.UTF_8.name()).getBytes();
    } catch (IOException e) {
        logger.error("writeCsv IOException:{}", e.getMessage(), e);
    } finally {
        try {
            if (csvPrinter != null) {
                csvPrinter.close();
            }
            if (bufferedWriter != null) {
                bufferedWriter.close();
            }
            if (outputStreamWriter != null) {
                outputStreamWriter.close();
            }
            if (byteArrayOutputStream != null) {
                byteArrayOutputStream.close();
            }
        } catch (IOException e) {
            logger.error("iostream close IOException:{}", e.getMessage(), e);
        }
    }
    return bytes;
}
```

### 5.4 AES256Utils

```java
public class AES256Utils {

    /**
     * 加密算法
     */
    private static final String DEFAULT_CIPHER_ALGORITHM = "AES/ECB/PKCS5Padding";

    private static final String KEY_ALGORITHM = "AES";


    private static byte[] base64decoder(byte[] src){
        return src.length == 0 ? src : Base64.getDecoder().decode(src);
    }

    /**
     * AES 解密操作
     *
     * @param content     待解密内容
     * @param aesPassword UUID和用户名的结果(密钥)
     * @return 返回Base64转码后的加密数据
     */
    public static String decrypt(String content, String aesPassword) {
        try {
            //实例化
            Cipher cipher = Cipher.getInstance(DEFAULT_CIPHER_ALGORITHM);
            //使用密钥初始化，设置为解密模式
            //使用项目密钥解密
            cipher.init(Cipher.DECRYPT_MODE, getSecretKey(aesPassword));
            //执行操作
            byte[] result = cipher.doFinal(base64decoder(content.getBytes()));
            return new String(result, StandardCharsets.UTF_8);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return null;
    }

    /**
     * 生成加密秘钥
     *
     * @return
     */
    private static SecretKeySpec getSecretKey(String key) {
        //返回生成指定算法密钥生成器的 KeyGenerator 对象
        KeyGenerator keyGenerator = null;
        try {
            keyGenerator = KeyGenerator.getInstance(KEY_ALGORITHM);

            // 解决操作系统内部状态不一致问题（部分liunx不指定类型，无法解密）
            SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
            secureRandom.setSeed(key.getBytes());
            keyGenerator.init(256, secureRandom);

            //生成一个密钥
            SecretKey secretKey = keyGenerator.generateKey();
            return new SecretKeySpec(secretKey.getEncoded(), KEY_ALGORITHM);// 转换为AES专用密钥
        } catch (NoSuchAlgorithmException ex) {
            ex.printStackTrace();
        }
        return null;
    }
}
```



### 5.5 雪花IdWorker

```java
public class IdWorker {
	// ==============================Fields===========================================
    /**
     * 开始时间截 (2015-01-01)
     */
    private static long startTime;

    /**
     * 机器id所占的位数
     */
    private final long workerIdBits = 5L;

    /**
     * 数据标识id所占的位数
     */
    private final long datacenterIdBits = 5L;

    /**
     * 支持的最大机器id，结果是31 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数)
     */
    private final long maxWorkerId = -1L ^ (-1L << workerIdBits);

    /**
     * 支持的最大数据标识id，结果是31
     */
    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);

    /**
     * 序列在id中占的位数
     */
    private final long sequenceBits = 12L;

    /**
     * 机器ID向左移12位
     */
    private final long workerIdShift = sequenceBits;

    /**
     * 数据标识id向左移17位(12+5)
     */
    private final long datacenterIdShift = sequenceBits + workerIdBits;

    /**
     * 时间截向左移22位(5+5+12)
     */
    private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;

    /**
     * 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095)
     */
    private final long sequenceMask = -1L ^ (-1L << sequenceBits);

    /**
     * 工作机器ID(0~31)
     */
    private long workerId;

    /**
     * 数据中心ID(0~31)
     */
    private long datacenterId;

    /**
     * 毫秒内序列(0~4095)
     */
    private long sequence = 0L;

    /**
     * 上次生成ID的时间截
     */
    private long lastTimestamp = -1L;

    //==============================Constructors=====================================


    static {
        String startDate = "2018-01-01 00:00:00";
        DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        LocalDateTime localDateTime = LocalDateTime.parse(startDate, df);
        startTime = localDateTime.toInstant(ZoneOffset.of("+8")).toEpochMilli();

    }


    /**
     * 构造函数
     *
     * @param workerId     工作ID (0~31)
     * @param datacenterId 数据中心ID (0~31)
     */
    public IdWorker(long workerId, long datacenterId) {
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
    }

    // ==============================Methods==========================================

    /**
     * 获得下一个ID (该方法是线程安全的)
     *
     * @return SnowflakeId
     */
    public synchronized long nextId() {
        long timestamp = timeGen();

        //如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(
                String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }

        //如果是同一时间生成的，则进行毫秒内序列
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            //毫秒内序列溢出
            if (sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                timestamp = tilNextMillis(lastTimestamp);
            }
        }
        //时间戳改变，毫秒内序列重置
        else {
            sequence = 0L;
        }

        //上次生成ID的时间截
        lastTimestamp = timestamp;

        //移位并通过或运算拼到一起组成64位的ID
        return ((timestamp - startTime) << timestampLeftShift) //
            | (datacenterId << datacenterIdShift) //
            | (workerId << workerIdShift) //
            | sequence;
    }

    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     *
     * @param lastTimestamp 上次生成ID的时间截
     * @return 当前时间戳
     */
    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }

    /**
     * 返回以毫秒为单位的当前时间
     *
     * @return 当前时间(毫秒)
     */
    protected long timeGen() {
        return System.currentTimeMillis();
    }
}
```

### 5.6 jsonUtil

```java
public class JsonUtils {

    // 定义jackson对象
    private static final ObjectMapper MAPPER = new ObjectMapper();

    /**
     * 将对象转换成json字符串。
     * <p>Title: pojoToJson</p>
     * <p>Description: </p>
     *
     * @param data Object
     * @return String/Json
     */
    public static String objectToJson(Object data) {
        try {
            String string = MAPPER.writeValueAsString(data);
            return string;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 将json结果集转化为对象
     *
     * @param jsonData json数据
     * @param clazz    对象中的object类型
     * @return pojo对象
     */
    public static <T> T jsonToPojo(String jsonData, Class<T> beanType) {
        try {
            T t = MAPPER.readValue(jsonData, beanType);
            return t;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 将json数据转换成pojo对象list
     * <p>Title: jsonToList</p>
     * <p>Description: </p>
     *
     * @param jsonData
     * @param beanType
     * @return
     */
    public static <T> List<T> jsonToList(String jsonData, Class<T> beanType) {
        JavaType javaType = MAPPER.getTypeFactory().constructParametricType(List.class, beanType);
        try {
            List<T> list = MAPPER.readValue(jsonData, javaType);
            return list;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }


    public static Object getFieldValueByName(Object obj, String fieldName) {
        try {
            // 获取obj类的字节文件对象
            Class c = obj.getClass();
            // 获取该类的成员变量
            Field f = c.getDeclaredField(fieldName);
            // 取消语言访问检查
            f.setAccessible(true);
            return f.get(obj);

        } catch (Exception e) {
        }
        return null;
    }

    /**
     * 根据属性名称赋值
     *
     * @param obj
     * @param fieldName
     * @param value
     */
    public static void setFieldValueByName(Object obj, String fieldName, Object value) {
        try {
            // 获取obj类的字节文件对象
            Class c = obj.getClass();
            // 获取该类的成员变量
            Field f = c.getDeclaredField(fieldName);
            // 取消语言访问检查
            f.setAccessible(true);
            Type type = f.getGenericType();
            // 给变量赋值
            if (type.equals(BigDecimal.class)) {
                f.set(obj, new BigDecimal(value.toString()));
            } else if (type.equals(Integer.class)) {
                f.set(obj, new Integer(value.toString()));
            } else if (type.equals(LocalDateTime.class)) {
                f.set(obj, LocalDateTime.parse(value.toString()));
            } else {
                f.set(obj, value);
            }
        } catch (Exception e) {
        }

    }
}

```

### 5.7 MD5Util

```java
public class MD5Util {

    private static final String[] hexDigits = new String[]{"0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "a", "b", "c", "d", "e", "f"};

    private static String byteArrayToHexString(byte[] b) {
        StringBuffer resultSb = new StringBuffer();

        for (int i = 0; i < b.length; ++i) {
            resultSb.append(byteToHexString(b[i]));
        }

        return resultSb.toString();
    }

    private static String byteToHexString(byte b) {
        int n = b;
        if (b < 0) {
            n = b + 256;
        }

        int d1 = n / 16;
        int d2 = n % 16;
        return hexDigits[d1] + hexDigits[d2];
    }

    public static String MD5Encode(String origin, String charsetname) {
        String resultString = null;

        try {
            resultString = new String(origin);
            MessageDigest md = MessageDigest.getInstance("MD5");
            if (charsetname != null && !"".equals(charsetname)) {
                resultString = byteArrayToHexString(md.digest(resultString.getBytes(charsetname)));
            } else {
                resultString = byteArrayToHexString(md.digest(resultString.getBytes()));
            }
        } catch (Exception var4) {
            ;
        }

        return resultString;
    }
}

```

### 5.8 RSAUtil

```java
public class RSAUtils {

    private static Map<Integer, String> keyMap = new HashMap<>();  //用于封装随机产生的公钥与私钥

    /**
     * 随机生成密钥对
     *
     * @throws NoSuchAlgorithmException
     */
    public static void genKeyPair() throws NoSuchAlgorithmException {
        // KeyPairGenerator类用于生成公钥和私钥对，基于RSA算法生成对象
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
        // 初始化密钥对生成器，密钥大小为96-1024位
        keyPairGen.initialize(1024, new SecureRandom());
        // 生成一个密钥对，保存在keyPair中
        KeyPair keyPair = keyPairGen.generateKeyPair();
        // 得到私钥
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
        // 得到公钥
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        String publicKeyString = new String(Base64.encodeBase64(publicKey.getEncoded()));
        // 得到私钥字符串
        String privateKeyString = new String(Base64.encodeBase64((privateKey.getEncoded())));
        // 将公钥和私钥保存到Map
        //0表示公钥
        keyMap.put(0, publicKeyString);
        //1表示私钥
        keyMap.put(1, privateKeyString);
    }

    /**
     * RSA公钥加密
     *
     * @param str       加密字符串
     * @param publicKey 公钥
     * @return 密文
     * @throws Exception 加密过程中的异常信息
     */
    public static String encrypt(String str, String publicKey) throws Exception {
        //base64编码的公钥
        byte[] decoded = Base64.decodeBase64(publicKey);
        RSAPublicKey pubKey = (RSAPublicKey) KeyFactory.getInstance("RSA").generatePublic(new X509EncodedKeySpec(decoded));
        //RSA加密
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, pubKey);
        String outStr = Base64.encodeBase64String(cipher.doFinal(str.getBytes("UTF-8")));
        return outStr;
    }

    /**
     * RSA私钥解密
     *
     * @param str        加密字符串
     * @param privateKey 私钥
     * @return 铭文
     * @throws Exception 解密过程中的异常信息
     */
    public static String decrypt(String str, String privateKey) throws Exception {
        //64位解码加密后的字符串
        byte[] inputByte = Base64.decodeBase64(str.getBytes("UTF-8"));
        //base64编码的私钥
        byte[] decoded = Base64.decodeBase64(privateKey);
        RSAPrivateKey priKey = (RSAPrivateKey) KeyFactory.getInstance("RSA").generatePrivate(new PKCS8EncodedKeySpec(decoded));
        //RSA解密
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, priKey);
        String outStr = new String(cipher.doFinal(inputByte));
        return outStr;
    }
}

```



 ### 5.9 JWTUtil

```java
public class JwtUtils {
    /**
     * 私钥加密token
     *
     * @param map           载荷中的数据
     * @param expireMinutes 过期时间，单位秒
     * @return
     * @throws Exception
     */
    public static String generateToken(Map<String, Object> map, PrivateKey key, int expireMinutes) throws Exception {
        return Jwts.builder()
                .setClaims(map)
                .setExpiration(DateTime.now().plusMinutes(expireMinutes).toDate())
                .setIssuedAt(new Date())
                .signWith(key, SignatureAlgorithm.RS256)
                .compact();
    }

    /**
     * 公钥解析token
     *
     * @param token 用户请求中的token
     * @return
     * @throws Exception
     */
    private static Jws<Claims> parserToken(String token, PublicKey key) {
        return Jwts.parser().setSigningKey(key).parseClaimsJws(token);
    }

    /**
     * 获取token中的用户信息
     *
     * @param token 用户请求中的令牌
     * @return 用户信息
     * @throws Exception
     */
    public static Map<String, Object> getInfoFromToken(String token, PublicKey key) throws Exception {
        Jws<Claims> claimsJws = parserToken(token, key);
        return claimsJws.getBody();
    }
```

### 5.10 SftpUtil

```java
@Slf4j
public class SftpUtil {
    private ChannelSftp sftp = null;

    private Session session;
    /**
     * SFTP 登录用户名
     */
    private String username;
    /**
     * SFTP 登录密码
     */
    private String password;

    /**
     * SFTP 服务器地址IP地址
     */
    private String host;
    /**
     * SFTP 端口
     */
    private int port;

    private static final String EXCE_MSG = "no such file";

    /**
     * 构造基于密码认证的sftp对象
     */
    public SftpUtil(String username, String password, String host, int port) {
        this.username = username;
        this.password = password;
        this.host = host;
        this.port = port;
    }


    public SftpUtil() {
    }


    /**
     * 连接sftp服务器
     */
    public void login() {
        try {
            JSch jsch = new JSch();
            session = jsch.getSession(username, host, port);
            if (password != null) {
                session.setPassword(password);
            }
            Properties config = new Properties();
            config.put("StrictHostKeyChecking", "no");
            session.setConfig(config);
            session.connect();
            Channel channel = session.openChannel("sftp");
            channel.connect();
            sftp = (ChannelSftp) channel;
        } catch (JSchException e) {
            throw new BusinessException("An error was reported when connecting to SFTP server");
        }
    }

    /**
     * 关闭连接 server
     */
    public void logout() {
        if (sftp != null) {
            if (sftp.isConnected()) {
                sftp.disconnect();
            }
        }
        if (session != null) {
            if (session.isConnected()) {
                session.disconnect();
            }
        }
    }


    /**
     * 将输入流的数据上传到sftp作为文件。文件完整路径="/home/sftpuser"+directory
     *
     * @param directory 上传到该目录
     * @param file      上传的文件
     * @return
     */
    public String upload(String directory, MultipartFile file) {
        String sftpFileName = file.getOriginalFilename();
        InputStream input = null;
        try {
            input = file.getInputStream();
            // 文件名添加uuid，保证文件名唯一
            if (StringUtils.isEmpty(sftpFileName)) {
                throw new BusinessException("请选择要上传的文件");
            }
            String suffix = sftpFileName.substring(sftpFileName.indexOf("."));
            sftpFileName = sftpFileName.substring(0, sftpFileName.indexOf("."));
            String uuid = UUID.randomUUID().toString().replaceAll("-", "");
            sftpFileName = sftpFileName + uuid + suffix;

            //进入目录
            createDir(directory);
        } catch (IOException e) {
            log.error("IO异常", e);
        }
        try {
            //上传文件
            sftp.put(input, sftpFileName);
        } catch (SftpException e) {
            throw new BusinessException("上传失败");
        }
        return sftpFileName;
    }

    /**
     * 下载文件
     *
     * @param directory    下载目录
     * @param downloadFile 下载的文件名
     * @return 字节数组
     */
    public byte[] download(String directory, String downloadFile) throws SftpException, IOException {
        if (directory != null && !"".equals(directory)) {
            sftp.cd(directory);
        }
        InputStream is = sftp.get(downloadFile);

        byte[] fileData = IOUtils.toByteArray(is);

        return fileData;
    }

    /**
     * 下载文件
     *
     * @param directory    下载目录
     * @param downloadFile 下载的文件名
     * @return 字节数组
     */
    public InputStream downloadInputStream(String directory, String downloadFile) throws SftpException, IOException {
        if (directory != null && !"".equals(directory)) {
            sftp.cd(directory);
        }
        InputStream is = sftp.get(downloadFile);
        return is;
    }


    /**
     * 下载单个文件到本地服务器
     *
     * @param directory    下载目录
     * @param downloadFile 下载的文件
     * @param saveFile     存在本地的路径
     */
    public void download(String directory, String downloadFile,
                         String saveFile) {
        try {
            sftp.cd(directory); //下载目录
            sftp.get(downloadFile, saveFile); //下载的文件 存在本地的路径
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    /**
     * 删除文件
     *
     * @param directory  要删除文件所在目录
     * @param deleteFile 要删除的文件
     */
    public void delete(String directory, String deleteFile) {
        try {
            sftp.cd(directory);
            sftp.rm(deleteFile);
        } catch (SftpException e) {
            log.error("删除文件错误", e);
            throw new BusinessException("删除文件失败");
        }

    }


    /**
     * 创建目录
     *
     * @param createpath
     * @return
     */
    public boolean createDir(String createpath) {
        try {
            if (isDirExist(createpath)) {
                this.sftp.cd(createpath);
                return true;
            }
            String[] pathArry = createpath.split("/");
            StringBuffer filePath = new StringBuffer("/");
            for (String path : pathArry) {
                if ("".equals(path)) {
                    continue;
                }
                filePath.append(path + "/");
                if (isDirExist(filePath.toString())) {
                    sftp.cd(filePath.toString());
                } else {
                    // 建立目录
                    sftp.mkdir(filePath.toString());
                    // 进入并设置为当前目录
                    sftp.cd(filePath.toString());
                }

            }
            this.sftp.cd(createpath);
            return true;
        } catch (SftpException e) {
            e.printStackTrace();
        }
        return false;
    }

    /**
     * 判断目录是否存在
     *
     * @param directory
     * @return
     */
    public boolean isDirExist(String directory) {
        boolean isDirExistFlag = false;
        try {
            SftpATTRS sftpAttrs = sftp.lstat(directory);
            isDirExistFlag = true;
            return sftpAttrs.isDir();
        } catch (Exception e) {
            if (EXCE_MSG.equals(e.getMessage().toLowerCase())) {
                isDirExistFlag = false;
            }
        }
        return isDirExistFlag;
    }
}
```

## 六、远程调用
### 6.1 restTemplate

```java
String url = apiUrl + "/content-v1/content/Portal/url-scheme";
HttpHeaders httpHeaders = new HttpHeaders();
httpHeaders.add("Authorization", "Bearer " +authorization);
HttpEntity<JSONObject> requestEntity = new HttpEntity<>(jsonObject, httpHeaders);
ResponseEntity<String> responseEntity = restTemplate.exchange(url, HttpMethod.POST, requestEntity, String.class);
if (responseEntity.getStatusCode() == HttpStatus.OK) {
    //接口成功
    String body = responseEntity.getBody();
    String code = JSONObject.parseObject(body).getString("code");
	} else {
    throw new BusinessException("调用失败！");
	}  
}
```

## 七、EasyExcel

* 实体类

```java
@Data
public class UploadFileImport {

    @ExcelProperty("binCode")
    private String binCode;

    @ExcelProperty("articleId")
    private String articleId;
}
```

`实体类忽略某个字段注解  -- @ExcelIgnore`



* 上传Excel

```java
List<UploadFileImport> data = null;
try {
    data = EasyExcel.read(file.getInputStream()).head(UploadFileImport.class).sheet().doReadSync();
} catch (IOException e) {
    log.warn("上传excel文件异常信息：{}", e);
}
if (CollectionUtils.isEmpty(data)) {
    throw new BusinessException("The uploaded file is empty");
}
```

* 方法一：导出Excel

```java
List<ExportBulkcodePo> exportBulkcodePoList = fileDetails.stream().map(a -> {
    ExportBulkcodePo exportBulkcodePo = new ExportBulkcodePo();
    exportBulkcodePo.setReason(a.getReason());
    exportBulkcodePo.setArticleId(a.getArticleId());
    exportBulkcodePo.setBinCode(a.getBinCode());
    return exportBulkcodePo;
}).collect(Collectors.toList());
response.setContentType("application/vnd.ms-excel");
response.setCharacterEncoding("utf-8");
response.setHeader("Access-Control-Expose-Headers", "fileName");
response.setHeader("filename", URLEncoder.encode(fileName, "UTF-8"));
EasyExcel.write(response.getOutputStream(), ExportBulkcodePo.class).sheet("binCode statistics export").doWrite(exportBulkcodePoList);

```

* 方法二：动态导出Excel

```java
InputStream templateFileName = this.getClass().getResourceAsStream("/template/汽油统计表模板.xlsx");
response.setContentType("application/vnd.ms-excel");
response.setCharacterEncoding("utf-8");
response.setHeader("Access-Control-Expose-Headers", "fileName");
response.setHeader("filename", URLEncoder.encode("能源消耗汇总统计", "UTF-8"));
ExcelWriter excelWriter = EasyExcel.write(response.getOutputStream()).withTemplate(templateFileName).build();
WriteSheet writeSheet = EasyExcel.writerSheet(0, "能源消耗汇总统计").build();
int i = 0;
for (int j = startYear; j <= endYear1; j++) {
    WriteTable table = EasyExcel.writerTable(i).needHead(true).head(TableHeadUtils.createHead(j)).build();
    String s = String.valueOf(j);
    List<CommonExport_2> collect = commonExport_2List.stream().filter(a -> a.getYear().equals(s)).collect(Collectors.toList());
    excelWriter.write(collect, writeSheet, table);
    i += typeList.size() * 2;
}
excelWriter.finish();

public static List<List<String>> createHead(int year){
    int[] array = {1,2,3,4,5,6,7,8,9,10,11,12,13,14};
    // 模型上没有注解，表头数据动态传入
    List<List<String>> head = new ArrayList<List<String>>();
    for (int i = 0; i < array.length; i++) {
        List<String> headCoulumn = new ArrayList<String>();
        if (i == 0) {
            headCoulumn.add(String.valueOf(year));
        } else if (i == array.length - 1) {
            headCoulumn.add("合计");
        } else {
            headCoulumn.add(i + "月");
        }
        head.add(headCoulumn);
    }
    return head;
}
```



dingding