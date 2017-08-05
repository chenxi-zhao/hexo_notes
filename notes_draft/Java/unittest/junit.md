#### 测试代码

```java
public class JUnitTest {

    /**
     * 只执行一次,在整个类执行之前执行
     */
    @BeforeClass
    public static void beforeClass() {
        System.out.println("in before class");
    }

    /**
     * 只执行一次,在整个类执行之后执行
     */
    @AfterClass
    public static void afterClass() {
        System.out.println("in after class");
    }

    /**
     * 每个测试方法被执行前都被执行一次
     */
    @Before
    public void before() {
        System.out.println("in before");
    }

    /**
     * 每个测试方法被执行后都被执行一次
     */
    @After
    public void after() {
        System.out.println("in after");
    }

    // test case 1
    @Test
    public void testCase1() {
        System.out.println("in test case 1");
    }

    // test case 2
    @Test
    public void testCase2() {
        System.out.println("in test case 2");
    }

    /**
     * 测试assertEquals
     */
    @Test
    public void testEquals() {
        Assert.assertEquals(1 + 2, 3);
    }

    /**
     * 测试assertTrue
     */
    @Test
    public void testTrue() {
        Assert.assertTrue(1 + 2 == 3);
    }

    /**
     * 测试assertFalse
     */
    @Test
    public void testFals() {
        Assert.assertFalse(1 + 2 == 4);
    }

    /**
     * 测试assertNotNull
     */
    @Test
    public void assertNotNull() {
        Assert.assertNotNull("not null");
    }

    /**
     * 测试assertNull
     */
    @Test
    public void assertNull() {
        Assert.assertNull(null);
    }

    /**
     * 测试fail和Ignore
     */
    @Test
    @Ignore
    public void assertFail() {
        Assert.fail();
    }

    /**
     * 测试异常
     */
    @Test(expected = NullPointerException.class)
    public void testException() {
        throw new NullPointerException();
    }

    /**
     * 测试时间
     */
    @Test(timeout = 1000)
    public void testTimeoutSuccess() {
        // do nothing
    }

    /**
     * 测试时间
     */
    @Test(timeout = 1000)
    public void testTimeoutFailed() {
        while (true) {

        }
    }
}
```
