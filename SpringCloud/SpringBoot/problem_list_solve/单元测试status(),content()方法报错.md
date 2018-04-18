[springboot测试的时候status(),content()方法报错](https://blog.csdn.net/qq_21549989/article/details/78873229)
----------------------
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloTests {

  
    private MockMvc mvc;

    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.standaloneSetup(new HelloworldController()).build();
    }

    @Test
    public void getHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/henry").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("Hello World")));
    }

}
```

是因为静态导入的原因，只要导入即可解决：

```java
import static org.hamcrest.Matchers.equalTo;    
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;    
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status; 
```
