# 快速入门

## 什么是Spring AI

Spring AI 旨在简化包含 AI 功能的应用程序的开发，避免不必要的复杂度。其核心功能就是将应用程序核心功能与大模型连接起来。

## 搭建工程

导入依赖
```xml
<dependencyManagement>  
    <dependencies>        <!-- Spring AI BOM -->  
        <dependency>  
            <groupId>org.springframework.ai</groupId>  
            <artifactId>spring-ai-bom</artifactId>  
            <version>${spring-ai.version}</version>  
            <type>pom</type>  
            <scope>import</scope>  
        </dependency>    
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.ai</groupId>  
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>  
    </dependency>
</dependencies>
```

编写配置
- openai
```yaml
spring:  
  ai:  
    openai:  
      base-url: https://api.chatanywhere.tech  
      api-key: OPEN_API_KEY  
      chat:  
        options:  
          model: gpt-3.5-turbo
```
- qwen
```yaml
spring:  
  ai:  
    openai:  
      base-url: https://dashscope.aliyuncs.com/compatible-mode  
      api-key: sk-cc9613f9abfe490dac47e425f4be7ce3  
      chat:  
        options:  
          model: qwen-plus
```

## 普通聊天

### 构造ChatClient

```java
@Configuration  
public class SpringAIConfig {  
    @Bean  
    public ChatClient chatClient(ChatClient.Builder builder) {  
        return builder.build();  
    }  
}
```

### 编写ChatService

```java
@Service  
@Slf4j  
@RequiredArgsConstructor  
public class IChatService implements ChatService {  
    private final ChatClient chatClient;  
    @Override  
    public String chat(String question) {  
        String content = chatClient.prompt()  
                .user(question)  
                .call()  
                .content();  
        log.info("question: {}, content: {}", question, content);  
        return content;  
    }  
}
```

### 编写Controller

```java
@RestController  
@RequiredArgsConstructor  
@RequestMapping("/chat")  
public class ChatController {  
    private final ChatService chatService;  
    @PostMapping  
    public String chat(String question) {  
        return chatService.chat(question);  
    }  
}
```

## 流式聊天

### SSE

在大模型的流式聊天模式中，数据是从服务端推送到客户端的，这就需要使用 Server-Sent Events，SSE 协议。其核心特征有：
- **单向通信**：仅服务端向客户端发送数据，客户端通过普通 HTTP 请求建立连接后等待推送。
- **基于 HTTP**：无需额外协议，兼容现有 HTTP 基础设施，如身份验证、CORS等。
- **自动重连**：客户端在连接断开时会自动尝试重新连接，支持自定义重试时间。
- **轻量级**：数据格式简单，开销低，适合高频词小数据量场景。
- **事件驱动**：支持定义不同事件类型，客户端可按需监听。

### Service 与 Controller

**Service**
```java
@Override  
public Flux<String> chatStream(String question) {  
    return chatClient.prompt()  
            .user(question)  
            .stream()  
            .content()  
            .doOnNext(content -> log.info("question: {}, content: {}", question, content))  
            .concatWith(Flux.just("[END]"));  
}
```

**Controller**
```java
@PostMapping(value = "stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)  
public Flux<String> chatStream(String question) {  
    return chatService.chatStream(question);  
}
```

# System角色设定

对于大模型的 System 角色设定，有两种方式，一种是局部设定，另一种是默认设定。如果系统⻆⾊的内容相对固定，在整个项⽬中使⽤同⼀个⻆⾊内容，就使⽤默认设定；反之，选择局部设定。 

如果在内容中设定了参数，但是在发起请求时并没有给对应的所有参数都提供参数值，则会报错。

## 局部设定

通过 `.system()` 方法设定系统角色
```java
@Override  
public String chat(String question) {  
    String content = chatClient.prompt()  
            .system(Constant.SYSTEM_ROLE)  
            .user(question)  
            .call()  
            .content();  
    log.info("question: {}, content: {}", question, content);  
    return content;  
}
```

## 默认设定

在 config 中，通过 `defaultSystem()` 方法设置默认系统角色。
```java
@Bean  
public ChatClient chatClient(ChatClient.Builder builder) {  
    return builder  
            .defaultSystem(Constant.SYSTEM_ROLE)  
            .build();  
}
```

## 动态参数

对于某些一直在变化的参数，可以在模型的系统角色设置中加入动态参数。在系统角色内容中，使用 `{}` 标记参数。

**示例**
```
# ⻆⾊  
你是 Java 开发助⼿，名字叫⼩智。  
当前时间是 {now}。
```

在 Service 中发起请求时，通过 `param()` 方法替换占位符，设置参数。这⾥设置的参数要与上述的占位符参数名称保持⼀致。
```java
@Override  
public String chat(String question) {  
    String content = chatClient.prompt()  
            .system(prompt -> prompt.param("now", DateTime.now()))  
            .user(question)  
            .call()  
            .content();  
    return content;  
}
```

# Advisors功能增强

Spring AI Advisors 提供了⼀种灵活且强⼤的⽅式，可以在Spring应⽤中轻松拦截、调整和增强基于AI的交互操作。

## 运行原理

在与⼤模型交互过程中，Advisors的执⾏流程如下：
1. 请求包装
	- 框架将用户输入的 Prompt 封装为 AdvisedRequest 对象。
	- 创建空的 AdvisorContext 上下文容器，用于链式传递处理状态。
2. Advisor 链预处理
	- 多个 Advisor 按链式顺序处理请求。
	- 每个 Advisor 都可以修改请求内容，或直接拦截请求并生成响应。
3. 模型调用
	- 框架内置的最终 Advisor 将标准化请求发送至大模型。
	- 触发 Chat Model 完成核心推理。
4. 相应处理
	- 模型输出通过 AdvisorContext 携带上下文原路返回。
	- 各个 Advisor 都可以二次处理响应，如 格式化输出结构、添加解释性内容、执行最终安全校验等。
![[项目/天机学堂/Inbox/Pasted image 20260127155648.png]]

## ⽇志Advisor

在 Spring AI 中，提供了 SimpleLoggerAdvisor 用于记录 request 和 response 数据的 Advisor，用于调试与大模型的交互。使用时，需要先定义 SimpleLoggerAdvisor 对象，再在 ChatClient 中加入该对象，同时在配置中添加对应的日志配置。

**SimpleLoggerAdvisor**
```java
@Bean  
public Advisor simpleLoggerAdvisor() {  
    return new SimpleLoggerAdvisor();  
}
```

在 ChatClient 中加入 SimpleLoggerAdvisor。
```java
@Bean  
public ChatClient chatClient(ChatClient.Builder builder,  
                             SimpleLoggerAdvisor simpleLoggerAdvisor) {  
    return builder  
            .defaultAdvisors(simpleLoggerAdvisor)  
            .defaultSystem(Constant.SYSTEM_ROLE)  
            .build();  
}
```

在日志中添加日志配置。
```yaml
logging:  
  level:  
    org:  
      springframework:  
        ai:  
          chat:  
            client:  
              advisor: DEBUG
```

## 会话记忆

在与大模型进行交流时，大模型是无状态的，如果想要再次有状态，就需要把之前的对话内容一起发送给大模型。在 SpringAI 中，已经对聊天记录的收集做了实现，只需要简单配置即可使用。

目前实现方式有三种，分别是：
- MessageChatMemoryAdvisor：将历史消息与当前用户消息进行合并，发给大模型。
- PromptChatMemoryAdvisor：将历史消息与系统提示词进行合并，发给大模型。
- VectorStoreChatMemoryAdvisor：将历史消息存储到向量数据库中，以实现**长期记忆**而非前面两种情况的短期记忆。短期记忆受限于上下文窗口长度，只能携带最近的聊天记录。存储在向量数据库中的历史消息默认也是放在系统提示词中。

如果模型支持，则优先使用 MessageChatMemoryAdvisor，否则再换成 PromptChatMemoryAdvisor。整合到系统提示词的方式兼容性较好。

### MessageChatMemoryAdvisor

该存储方式是基于内存存储，服务重启后，聊天记录将丢失，所以这种⽅式不适合⽤在真实项⽬中。如果真实项目中希望可以长期保存聊天记录，可选择使⽤数据库、Redis等⽅式进⾏存储。

**配置ChatMemory存储器**
```java
@Bean  
public ChatMemory chatMemory() {  
    return new InMemoryChatMemory();  
}
```

**配置MessageChatMemoryAdvisor**
```java
@Bean  
public Advisor messageChatMemoryAdvisor(ChatMemory chatMemory) {  
    return new MessageChatMemoryAdvisor(chatMemory);  
}
```

**添加配置好的Advisor**
```java
@Bean  
public ChatClient chatClient(ChatClient.Builder builder,  
                             Advisor simpleLoggerAdvisor,  
                             Advisor messageChatMemoryAdvisor) {  
    return builder  
            .defaultAdvisors(simpleLoggerAdvisor, messageChatMemoryAdvisor)  
            .defaultSystem(Constant.SYSTEM_ROLE)  
            .build();  
}
```

### PromptChatMemoryAdvisor

类似上文的 MessageChatMemoryAdvisor，只需要先定义，然后加入 defaultAdvisors 即可。

**定义**
```java
@Bean Advisor promptChatMemoryAdvisor(ChatMemory chatMemory) {  
    return new PromptChatMemoryAdvisor(chatMemory);  
}
```

**加入**
```java
@Bean  
public ChatClient chatClient(ChatClient.Builder builder,  
                             Advisor simpleLoggerAdvisor,  
                             Advisor promptChatMemoryAdvisor) {  
    return builder  
            .defaultAdvisors(simpleLoggerAdvisor, promptChatMemoryAdvisor)  
            .defaultSystem(Constant.SYSTEM_ROLE)  
            .build();  
}
```

### 会话id

到目前为止，已经知道了如何设置会话记忆。但是这种方式不同用户的会话记录都存储在同一套记忆中，为使每个用户都有自己的记忆，需要一个会话id对其进行区分。在 SpringAI 中，提供了通过 sessionId 区分用户的功能。

**定义chatDTO**
```java
@Data  
@Builder  
@NoArgsConstructor  
@AllArgsConstructor  
public class ChatDTO {  
    private String question;  
    private String sessionId;  
}
```

**修改Service**
- 关键部分在于 `.advisor()` 中的内容
```java
@Override  
public String chat(String question, String sessionId) {  
    String content = chatClient.prompt()  
            .system(prompt -> prompt.param("now", DateTime.now()))  
            .advisors(advisorSpec -> advisorSpec.param(AbstractChatMemoryAdvisor.CHAT_MEMORY_CONVERSATION_ID_KEY, sessionId))  
            .user(question)  
            .call()  
            .content();  
    log.info("question: {}, content: {}", question, content);  
    return content;  
}
```

**修改Controller**
```java
@PostMapping  
public String chat(@RequestBody ChatDTO chatDTO) {  
    return chatService.chat(chatDTO.getQuestion(), chatDTO.getSessionId());  
}
```

## 敏感词校验

Spring AI 中提供了安全组件 SafeGuardAdvisor，当用户输入包含敏感词时，立即拦截请求。避免大模型处理，既节省了计算资源，又避免了安全风险。

**定义SafeGuardAdvisor**
```java
@Bean  
public Advisor safeGuardAdvisor() {  
    // 敏感词列表  
    List<String> sensitiveWords = List.of("海绵宝宝", "派大星");  
    return new SafeGuardAdvisor(sensitiveWords,  
            "敏感词提示: 请勿输入敏感词",  
            Advisor.DEFAULT_CHAT_MEMORY_PRECEDENCE_ORDER);  
}
```

**添加Advisor到ChatClient**
- safeGuardAdvisor 是负责处理敏感词的 Advisor。
```java
@Bean  
public ChatClient chatClient(ChatClient.Builder builder,  
                             Advisor simpleLoggerAdvisor,  
                             Advisor messageChatMemoryAdvisor,  
                             Advisor safeGuardAdvisor) {  
    return builder  
            .defaultAdvisors(simpleLoggerAdvisor, messageChatMemoryAdvisor, safeGuardAdvisor)  
            .defaultSystem(Constant.SYSTEM_ROLE)  
            .build();  
}
```

# Tool Calling

## 运行原理

Spring AI 提供了 Tool Calling 的方式增强大模型，通过这种方式可以与外部系统或者其他微服务系统整合起来。

**流程说明**
- 定义工具：在聊天请求中声明工具信息，包括：名称、功能描述、输入参数格式等。
- 模型发起调用：若模型需要使用工具，则返回工具名称和符合预定义格式的输入参数。
- 执行工具：应用程序根据工具名称匹配具体工具，并传递输入参数执行操作。
- 处理结果：应用程序接收工具执行结果，进行必要的数据处理。
- 返回模型：将工具调用结果发送给模型，作为生成最终回复的上下文依据。
- 生成最终响应：模型结合工具返回的结果，输出完整的回答内容。
![[项目/天机学堂/Inbox/Pasted image 20260128183235.png]]

## 定义DTO

定义一个 DTO，存储查询到的天气信息。
```java
@Data  
@Builder  
@NoArgsConstructor  
@AllArgsConstructor  
public class WeatherDTO {  
  
    @JsonPropertyDescription("城市ID")  
    private String cityId;  
    @JsonPropertyDescription("城市名称")  
    private String city;  
    @JsonPropertyDescription("当前温度(单位:℃)")  
    private String temperature;  
    @JsonPropertyDescription("低温(单位:℃)")  
    private String lowTemperature;  
    @JsonPropertyDescription("高温(单位:℃)")  
    private String highTemperature;  
    @JsonPropertyDescription("数据日期(格式:YYYYMMDD)")  
    private String date;  
    @JsonPropertyDescription("空气质量指数")  
    private String quality;  
    @JsonPropertyDescription("PM2.5浓度(单位:微克/立方米)")  
    private Double pm25;  
  
}
```

## 定义Tool

定义 Tool 并将其注册成为组件，使其有能力根据城市id查询天气信息。这种方式使用模拟数据模拟查询天气预报的过程，如果需要真实天气信息，则需要引入天气预报 API。
- @Tool 的作用是指定方法作为一个工具，通过 description 属性描述这个工具的作用
- @ToolParam的作用是指定工具方法的入参，也可以是无参的，description 属性负责描述传入参数的含义

```java
@Component  
public class WeatherTools {  
    @Tool(description = "根据城市id查询天气信息")  
    public WeatherDTO getWeather(@ToolParam(description = "城市id") String cityId) {  
        return WeatherDTO.builder()  
                .cityId(cityId)  
                .city("北京")  
                .temperature("25") // 当前温度  
                .lowTemperature("20")// 低温  
                .highTemperature("30")// 高温  
                .date("2023-10-01")// 数据日期  
                .quality("良")// 空气质量  
                .pm25(15.5)// PM2.5数值  
                .build();  
    }  
}
```

## 注册Tool

通过 `.defaultTools()` 方法为 ChatClient 添加默认工具。
```java
@Bean  
public ChatClient chatClient(ChatClient.Builder builder,  
                             Advisor simpleLoggerAdvisor,  
                             Advisor messageChatMemoryAdvisor,  
                             Advisor safeGuardAdvisor) {  
    return builder  
            .defaultAdvisors(simpleLoggerAdvisor, messageChatMemoryAdvisor, safeGuardAdvisor)  
            .defaultSystem(Constant.SYSTEM_ROLE)  
            .defaultTools(new WeatherTools())  
            .build();  
}
```

## 优化

为了能够获取真实可信的天气预报信息，引入天气预报 API，通过对应网址和城市代码就可以查询到指定城市的天气信息。同时，为了支持查询不同城市名称的天气数据，可以将城市名与城市id的对应列表添加到系统提示词中，使大模型可以找到城市对应的城市id。

```java
@Component  
public class WeatherTools {  
    @Tool(description = "根据城市id查询天气信息")  
    public WeatherDTO getWeather(@ToolParam(description = "城市id") String cityId) {  
        // 通过 http 请求获取天气信息  
        String url = "http://t.weather.itboy.net/api/weather/city/" + cityId;  
        String data = HttpUtil.get(url);  
        JSONObject jsonObject = JSONUtil.parseObj(data);  
  
        return WeatherDTO.builder()  
                .cityId(jsonObject.getByPath("cityInfo.citykey", String.class)) // 城市ID  
                .city(jsonObject.getByPath("cityInfo.city", String.class))      // 城市名称  
                .date(jsonObject.getByPath("date", String.class))               // 数据日期  
                .temperature(jsonObject.getByPath("data.wendu", String.class))  // 当前温度  
                .lowTemperature(jsonObject.getByPath("data.forecast[0].low", String.class))     // 低温  
                .highTemperature(jsonObject.getByPath("data.forecast[0].high", String.class))   // 高温  
                .quality(jsonObject.getByPath("data.quality", String.class))    // 空气质量  
                .pm25(jsonObject.getByPath("data.pm25", Double.class))          // PM2.5数值  
                .build();  
    }
}
```

# RAG检索增强

## 需求分析

在天气查询案例中，将城市名和id的对应关系添加到了系统提示词中。但是当城市信息的数据量比较大时，就不适合再放到提示词中，此时就适合使用 RAG 检索增强方案。

所以说可以将列表信息提前添加到知识库中，请求发送到大模型后，再通过知识库查询城市以及对应的id，并将数据和用户请求一并发送给大模型。如此就不需要在系统提示词中加入大量数据。

## RAG原理

### 执行流程

RAG 中分为 **文档提取 (ETL)** 和 **检索增强生成 (RAG)** 两个核心流程，分别在离线和在线条件下进行处理。

**文档提取 (ETL)** - 将非结构化文档转为结构化、可检索的向量数据
- 从数据源中读取原始文档。
- 通过分割模块 (`<<Split>>`)，将文档切分为更小的数据块 (chunks)。
- 通过转换模块处理数据块，将数据向量化、添加元数据等。
- 将处理好的数据库写入向量数据库。

**检索增强生成 (RAG)** - 通过外部数据库提升生成结果的准确性
- 接收用户提问
- 从向量数据库中检索与查询最相关的数据块 `<<Retrieve>>`
- 将检索到的上下文信息与用户问题融合，生成增强后的数据 `<<Augment>>`
- 通过聊天模型生成回答。

![[项目/天机学堂/Inbox/Pasted image 20260128204237.png]]

### 相似度计算

通常，先将数据进行向量化。计算机无法直接理解文本、图片等数据，所以需要向量化的过程，将非结构化数据转为计算机可识别的数字。

随后，通过相似度算法进行判断。余弦相似度算法是通过计算两个向量在多维空间中夹角的余弦值评估他们的相似度，相似度取值范围是 `[-1, 1]`，余弦值越接近1，说明两个向量越相似；欧氏距离算法通过衡量多维空间中两点间直线建立评估相似度，距离值越小越相似。

### 向量数据库

Spring AI 支持多种向量数据库，常见有：
- ElasticSearch Vector Store
- MongoDB Atlas Vector Store
- Redis Vector Store
- Simple Vector Store

## 城市数据知识库

### 数据向量化

**配置开启向量化模型**
```yaml
spring:  
  ai:  
    openai:  
      embedding:  
        options:  
          model: text-embedding-v3  
          dimensions: 1024
```

**创建内存向量数据库**
```java
@Bean  
public VectorStore vectorStore(EmbeddingModel embeddingModel) {  
    return SimpleVectorStore.builder(embeddingModel).build();  
}
```

### 存储数据

读取准备好的数据，将数据通过分割器切成小块，并存储到向量库中。
```java
@Component  
@Slf4j  
@RequiredArgsConstructor  
public class CityEmbedding {  
    private final VectorStore vectorStore;   
    @Value("classpath:citys.txt")  
    private Resource resource;  
    @PostConstruct  
    public void init() {  
        // 读取文档  
        TextReader textReader = new TextReader(resource);  
        textReader.getCustomMetadata().put("filename", "citys.txt");  
        // 拆分文档  
        List<Document> documentList = textReader.get();  
        TokenTextSplitter tokenTextSplitter = new TokenTextSplitter(200, 100, 120, 10000, false);  
        List<Document> splitDocuments = tokenTextSplitter.apply(documentList);  
        // 存入向量数据库  
        vectorStore.add(splitDocuments);  
        log.info("数据写入向量数据库成功, 条数为: {}", splitDocuments.size());  
    }  
}
```

### RAG实现

```java
@Override  
public String chat(String question, String sessionId) {  
    SearchRequest searchRequest = SearchRequest.builder()  
            .query(question)  
            .topK(3)  
            .build();  
    String content = chatClient.prompt()  
            .advisors(advisorSpec -> advisorSpec  
                    .advisors(new QuestionAnswerAdvisor(vectorStore, searchRequest))  
                    .param(AbstractChatMemoryAdvisor.CHAT_MEMORY_CONVERSATION_ID_KEY, sessionId))  
            .tools(new DateTimeTools())  
            .user(question)  
            .call()  
            .content();  
    log.info("question: {}, content: {}", question, content);  
    return content;  
}
```

