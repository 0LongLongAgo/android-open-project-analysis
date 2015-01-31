Java ��̬����
----------------
> ����Ϊ [Android ��Դ��Ŀʵ��ԭ�����](https://github.com/android-cn/android-open-project-analysis) �����������е� ��̬���� ����    
 ��Ŀ��ַ��[Jave Proxy](http://www.grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/6-b27/java/lang/reflect/Proxy.java#Proxy)�������İ汾��[openjdk 1.6](http://www.grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/6-b27/java/lang/reflect/Proxy.java#Proxy)��Demo ��ַ��[Proxy Demo](https://github.com/android-cn/android-open-project-demo/tree/master/java-dynamic-proxy)    
 �����ߣ�[Caij](https://github.com/Caij)��У���ߣ�[Trinea](https://github.com/Trinea)��У��״̬�����   

### 1. ��ظ���
(1). **����**����ĳЩ����£����ǲ�ϣ�����ǲ���ֱ�ӷ��ʶ��� A������ͨ������һ���н���� B���� B ȥ���� A ���Ŀ�ģ����ַ�ʽ���Ǿͳ�Ϊ����  
������� A ���������ǳ�Ϊί���࣬Ҳ��Ϊ�������࣬���� B �������Ϊ�����ࡣ  
�����ŵ��У�  
* ����ί�����ʵ��  
* ������ı�ί��������������һЩ���⴦��������ӳ�ʼ�жϼ�������������  

���ݳ�������ǰ�������Ƿ��Ѿ����ڣ����Խ������Ϊ��̬����Ͷ�̬����  

(2). **��̬����**���������ڳ�������ǰ�Ѿ����ڵĴ���ʽ��Ϊ��̬����  
ͨ��������Ϳ���֪�����ɿ�����Ա��д���Ǳ��������ɴ�����ķ�ʽ�����ھ�̬���������Ǽ򵥵ľ�̬����ʵ����  
```java
class ClassA {
    public void operateMethod1() {};

    public void operateMethod2() {};

    public void operateMethod3() {};
}

public class ClassB {
    private ClassA a;

    public ClassB(ClassA a) {
        this.a = a;
    }

    public void operateMethod1() {
        a.operateMethod1();
    };

    public void operateMethod2() {
        a.operateMethod2();
    };

    // not export operateMethod3()
}
```
����`ClassA`��ί���࣬`ClassB`�Ǵ����࣬`ClassB`�еĺ�������ֱ�ӵ���`ClassA`���󣬲���������`Class`��`operateMethod3()`������  

��̬�����д������ί����Ҳ�����̳�ͬһ�����ʵ��ͬһ�ӿڡ�  

(3). **��̬����**���������ڳ�������ǰ�����ڡ�����ʱ�ɳ���̬���ɵĴ���ʽ��Ϊ��̬����  
Java �ṩ�˶�̬�����ʵ�ַ�ʽ������������ʱ�̶�̬���ɴ����ࡣ���ִ���ʽ��һ��ô��Ǻܷ���Դ�����ĺ�����ͳһ�����⴦�����¼���к���ִ��ʱ�䡢���к���ִ��ǰ�����֤�жϡ���ĳ�����⺯�����������������������̬����ʽһ���޸�ÿ��������  

`��̬����`�Ƚϼ򵥣����Ĳ������ܣ��ص����`��̬����`��  

### 2. ��̬����ʵ��
#### ʵ�ֶ�̬�������������  
(1). �½�ί���ࣻ  
(2). ʵ��`InvocationHandler`�ӿڣ����Ǹ������Ӵ������ί������м������ʵ�ֵĽӿڣ�  
(3). ͨ��`Proxy`���½����������  

����ͨ��ʵ��������ܣ���������������ͳ��ĳ�������к�����ִ��ʱ�䣬��ͳ�ķ�ʽ�������ÿ������ǰ���ͳ�ƣ���̬����ʽ���£�  
#### 2.1 �½�ί����
```java
public interface Operate {

    public void operateMethod1();

    public void operateMethod2();

    public void operateMethod3();
}

public class OperateImpl implements Operate {

    @Override
    public void operateMethod1() {
        System.out.println("Invoke operateMethod1");
        sleep(110);
    }

    @Override
    public void operateMethod2() {
        System.out.println("Invoke operateMethod2");
        sleep(120);
    }

    @Override
    public void operateMethod3() {
        System.out.println("Invoke operateMethod3");
        sleep(130);
    }

    private static void sleep(long millSeconds) {
        try {
            Thread.sleep(millSeconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
`Operate`��һ���ӿڣ�������һЩ����������Ҫͳ����Щ������ִ��ʱ�䡣  
`OperateImpl`��ί���࣬ʵ��`Operate`�ӿڡ�ÿ������������ַ��������ȴ�һ��ʱ�䡣  
��̬����Ҫ��ί�������ʵ����ĳ���ӿڣ���������ί����`OperateImpl`ʵ����`Operate`��ԭ��������΢��������  

#### 2.2. ʵ�� InvocationHandler �ӿ�
```java
public class TimingInvocationHandler implements InvocationHandler {

    private Object target;

    public TimingInvocationHandler() {}

    public TimingInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        long start = System.currentTimeMillis();
        Object obj = method.invoke(target, args);
        System.out.println(method.getName() + " cost time is:" + (System.currentTimeMillis() - start));
        return obj;
    }
}
```
`target`���Ա�ʾί�������  

`InvocationHandler`ʵ���Ǹ������Ӵ������ί������м������ʵ�ֵĽӿڡ�����ֻ��һ��  
```java
public Object invoke(Object proxy, Method method, Object[] args)
```
������Ҫȥʵ�֣�������  
`proxy`��ʾ`2.3 ͨ�� Proxy.newProxyInstance() ���ɵĴ��������`��  
`method`��ʾ������󱻵��õĺ�����  
`args`��ʾ������󱻵��õĺ����Ĳ�����  

���ô�������ÿ������ʵ�����ն��ǵ�����`InvocationHandler`��`invoke`����������������`invoke`ʵ��������˿�ʼ������ʱ�����л�������ί�������`target`����Ӧ�����������������ͳ��ִ��ʱ�������  
`invoke`����������Ҳ����ͨ����`method`��һЩ�жϣ��Ӷ���ĳЩ�������⴦��  

#### 2.3. ͨ�� Proxy �ྲ̬�������ɴ������
```java
public class Main {
    public static void main(String[] args) {

        // create proxy instance
        TimingInvocationHandler timingInvocationHandler = new TimingInvocationHandler(new OperateImpl());
        Operate operate = (Operate)(Proxy.newProxyInstance(Operate.class.getClassLoader(), new Class[] {Operate.class},
                timingInvocationHandler));
        
        // call method of proxy instance
        operate.operateMethod1();
        System.out.println();
        operate.operateMethod2();
        System.out.println();
        operate.operateMethod3();
    }
}
```
���������Ƚ�ί�������`new OperateImpl()`��Ϊ`TimingInvocationHandler`���캯����δ���`timingInvocationHandler`����  
Ȼ��ͨ��`Proxy.newProxyInstance(��)`�����½���һ���������ʵ�ʴ������������ʱ��̬���ɵġ����ǵ��øô������ĺ����ͻ���õ�`timingInvocationHandler`��`invoke`����(�ǲ����е����ƾ�̬����)����`invoke`����ʵ���е���ί�������`new OperateImpl()`��Ӧ�� method(�ǲ����е����ƾ�̬����)��  

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```
`loader`��ʾ�������  
`interfaces`��ʾί����Ľӿڣ�����ί����ʱ��Ҫʵ����Щ�ӿ�  
`h`��`InvocationHandler`ʵ������󣬸������Ӵ������ί������м���  

���ǿ���������⣬���ϵĶ�̬����ʵ��ʵ����˫��ľ�̬�����������ṩ��ί���� B������̬�����˴����� A�������߻���Ҫ�ṩһ��ʵ����`InvocationHandler`������ C������ C ���Ӵ����� A ��ί���� B�����Ǵ����� A ��ί���࣬ί���� B �Ĵ����ࡣ�û�ֱ�ӵ��ô����� A �Ķ���A ������ת����ί���� C��ί���� C �ٽ�����ת��������ί���� B��  

### 3. ��̬����ԭ��
ʵ���������һ���Ѿ�˵���˶�̬���������ԭ����������ϸ������
#### 3.1 ���ɵĶ�̬���������
����������ʾ����������ʱ�Զ����ɵĶ�̬��������룬��εõ���Щ���ɵĴ������[ProxyUtils](https://github.com/android-cn/android-open-project-demo/blob/master/java-dynamic-proxy/src/com/codekk/java/test/dynamicproxy/util/ProxyUtils.java)���鿴 class �ļ���ʹ�� jd-gui  
```java
import com.codekk.java.test.dynamicproxy.Operate;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy
  implements Operate
{
  private static Method m4;
  private static Method m1;
  private static Method m5;
  private static Method m0;
  private static Method m3;
  private static Method m2;

  public $Proxy0(InvocationHandler paramInvocationHandler)
    throws 
  {
    super(paramInvocationHandler);
  }

  public final void operateMethod1()
    throws 
  {
    try
    {
      h.invoke(this, m4, null);
      return;
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }

  public final boolean equals(Object paramObject)
    throws 
  {
    try
    {
      return ((Boolean)h.invoke(this, m1, new Object[] { paramObject })).booleanValue();
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }

  public final void operateMethod2()
    throws 
  {
    try
    {
      h.invoke(this, m5, null);
      return;
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }

  public final int hashCode()
    throws 
  {
    try
    {
      return ((Integer)h.invoke(this, m0, null)).intValue();
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }

  public final void operateMethod3()
    throws 
  {
    try
    {
      h.invoke(this, m3, null);
      return;
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }

  public final String toString()
    throws 
  {
    try
    {
      return (String)h.invoke(this, m2, null);
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }

  static
  {
    try
    {
      m4 = Class.forName("com.codekk.java.test.dynamicproxy.Operate").getMethod("operateMethod1", new Class[0]);
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
      m5 = Class.forName("com.codekk.java.test.dynamicproxy.Operate").getMethod("operateMethod2", new Class[0]);
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      m3 = Class.forName("com.codekk.java.test.dynamicproxy.Operate").getMethod("operateMethod3", new Class[0]);
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      return;
    }
    catch (NoSuchMethodException localNoSuchMethodException)
    {
      throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
    }
    catch (ClassNotFoundException localClassNotFoundException)
    {
      throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
    }
  }
}
```  
�������ǿ��Կ�����̬���ɵĴ���������`com.sun.proxy`Ϊ������`$Proxy`Ϊ����ǰ׺���̳���`Proxy`������ʵ����`Proxy.newProxyInstance(��)`�ڶ���������������нӿڵ��ࡣ  
���е�`operateMethod1()`��`operateMethod2()`��`operateMethod3()`��������ֱ�ӽ���`h`ȥ����`h`�ڸ���`Proxy`�ж���Ϊ  
```java
protected InvocationHandler h;
```
��Ϊ`Proxy.newProxyInstance(��)`������������  
����`InvocationHandler`������ C ���Ӵ����� A ��ί���� B�����Ǵ����� A ��ί���࣬ί���� B �Ĵ����ࡣ  
  
#### 3.2. ���ɶ�̬������ԭ��
������� Java 1.6 Դ����з�������̬���������ڵ���`Proxy.newProxyInstance(��)`����ʱ���ɵġ�  
#### (1). newProxyInstance(��) 
�����������£�  
```java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
    throws IllegalArgumentException
{
    if (h == null) {
        throw new NullPointerException();
    }

    /*
     * Look up or generate the designated proxy class.
     */
    Class cl = getProxyClass(loader, interfaces);

    /*
     * Invoke its constructor with the designated invocation handler.
     */
    try {
        Constructor cons = cl.getConstructor(constructorParams);
        return (Object) cons.newInstance(new Object[] { h });
    } catch (NoSuchMethodException e) {
        throw new InternalError(e.toString());
    } catch (IllegalAccessException e) {
        throw new InternalError(e.toString());
    } catch (InstantiationException e) {
        throw new InternalError(e.toString());
    } catch (InvocationTargetException e) {
        throw new InternalError(e.toString());
    }
}
```
���п��Կ������ȵ���`getProxyClass(loader, interfaces)`�õ���̬�����࣬Ȼ��`InvocationHandler`��Ϊ�����๹�캯������½����������  

#### (2). getProxyClass(��) 
�������뼰��������(ʡ����ԭ��Ϊע��)�� 
```java
/**
 * �õ������࣬��������̬����
 * @param loader ���������� ClassLoader
 * @param interfaces ��������Ҫʵ�ֵĽӿ�
 * @return
 */
public static Class<?> getProxyClass(ClassLoader loader,
                                     Class<?>... interfaces)
    throws IllegalArgumentException
{
    if (interfaces.length > 65535) {
        throw new IllegalArgumentException("interface limit exceeded");
    }
    // �����������
    Class proxyClass = null;

    /* collect interface names to use as key for proxy class cache */
    String[] interfaceNames = new String[interfaces.length];

    Set interfaceSet = new HashSet();       // for detecting duplicates

    /**
     * ��� interfaces ���飬����������
     * ��1���Ƿ������ָ���� ClassLoader ��
     * ��2���Ƿ��� Interface
     * ��3���Ƿ� interfaces �����ظ�
     */
    for (int i = 0; i < interfaces.length; i++) {
        String interfaceName = interfaces[i].getName();
        Class interfaceClass = null;
        try {
            interfaceClass = Class.forName(interfaceName, false, loader);
        } catch (ClassNotFoundException e) {
        }
        if (interfaceClass != interfaces[i]) {
            throw new IllegalArgumentException(
                interfaces[i] + " is not visible from class loader");
        }

        if (!interfaceClass.isInterface()) {
            throw new IllegalArgumentException(
                interfaceClass.getName() + " is not an interface");
        }

        if (interfaceSet.contains(interfaceClass)) {
            throw new IllegalArgumentException(
                "repeated interface: " + interfaceClass.getName());
        }
        interfaceSet.add(interfaceClass);

        interfaceNames[i] = interfaceName;
    }

    // �Խӿ�����Ӧ�� List ��Ϊ����� key
    Object key = Arrays.asList(interfaceNames);

    /*
     * loaderToCache �Ǹ�˫��� Map����һ�� key Ϊ ClassLoader���ڶ��� key Ϊ ����� List��value Ϊ�������������
     */
    Map cache;
    synchronized (loaderToCache) {
        cache = (Map) loaderToCache.get(loader);
        if (cache == null) {
            cache = new HashMap();
            loaderToCache.put(loader, cache);
        }
    }

    /*
     * ������Ľӿ�����Ӧ�� List Ϊ key ���Ҵ����࣬������Ϊ��
     *     �����ã���ʾ�������Ѿ��ڻ�����
     *     pendingGenerationMarker ���󣬱�ʾ���������������У��ȴ�������ɷ��ء�
     *     null ��ʾ���ڻ�������û�п�ʼ���ɣ���ӱ�ǵ������д����ɴ�����
     */
    synchronized (cache) {
        do {
            Object value = cache.get(key);
            if (value instanceof Reference) {
                proxyClass = (Class) ((Reference) value).get();
            }
            if (proxyClass != null) {
                // proxy class already generated: return it
                return proxyClass;
            } else if (value == pendingGenerationMarker) {
                // proxy class being generated: wait for it
                try {
                    cache.wait();
                } catch (InterruptedException e) {
                }
                continue;
            } else {
                cache.put(key, pendingGenerationMarker);
                break;
            }
        } while (true);
    }

    try {
        String proxyPkg = null;     // package to define proxy class in

        /*
         * ��� interfaces �д��ڷ� public �Ľӿڣ������з� public �ӿڱ�����ͬһ�����棬�Һ������ɵĴ�����Ҳ��Ҫ�ڸ�����
         */
        for (int i = 0; i < interfaces.length; i++) {
            int flags = interfaces[i].getModifiers();
            if (!Modifier.isPublic(flags)) {
                String name = interfaces[i].getName();
                int n = name.lastIndexOf('.');
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {     // if no non-public proxy interfaces,
            proxyPkg = "";          // use the unnamed package
        }

        {
            // ���ɴ������������jdk 1.6 �汾��ȱ�ٶ�����������Ѿ����ڵĴ���
            long num;
            synchronized (nextUniqueNumberLock) {
                num = nextUniqueNumber++;
            }
            String proxyName = proxyPkg + proxyClassNamePrefix + num;

            // ��̬���ɴ�������ֽ���
            // ���յ��� sun.misc.ProxyGenerator.generateClassFile() �õ������������Ϣд�� DataOutputStream ʵ��
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces);
            try {
                // native ��ʵ�֣���������ش����ಢ�����������
                proxyClass = defineClass0(loader, proxyName,
                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
                throw new IllegalArgumentException(e.toString());
            }
        }
        // add to set of all generated proxy classes, for isProxyClass
        proxyClasses.put(proxyClass, null);

    } finally {
        // ���������ɳɹ��򱣴浽���棬����ӻ�����ɾ����Ȼ��֪ͨ�ȴ��ĵ���
        synchronized (cache) {
            if (proxyClass != null) {
                cache.put(key, new WeakReference(proxyClass));
            } else {
                cache.remove(key);
            }
            cache.notifyAll();
        }
    }
    return proxyClass;
}
```

**������Ҫ���������֣�**  
* ��� interfaces ���飬�����Ƿ������ָ���� ClassLoader �ڡ��Ƿ��� Interface��interfaces ���Ƿ����ظ�
* �ӿ�����Ӧ�� List Ϊ key ���Ҵ����࣬������Ϊ��
  * �����ã���ʾ�������Ѿ��ڻ����У�
  * pendingGenerationMarker ���󣬱�ʾ���������������У��ȴ�������ɷ��أ�
  * null ��ʾ���ڻ�������û�п�ʼ���ɣ���ӱ�ǵ������д����ɴ����ࡣ
* ��������಻���ڵ���`ProxyGenerator.generateProxyClass(��)`���ɴ����ಢ���뻺�棬֪ͨ�ڵȴ��Ļ��档  

**�����м���ע��ĵط���**  
* ������Ļ��� key Ϊ�ӿ�����Ӧ�� List���ӿ�˳��ͬ��ʾ��ͬ�� key ����ͬ�����ࡣ  
* ����������� ClassLoader ���Ѿ����ڵ����û��������  
* ��� interfaces �д��ڷ� public �Ľӿڣ������з� public �ӿڱ�����ͬһ�����棬�Һ������ɵĴ�����Ҳ��Ҫ�ڸ����档  
* ���Կ��� "sun.misc.ProxyGenerator.saveGeneratedFiles" ���أ����涯̬�ൽĿ�ĵ�ַ��  

Java 1.7 ��ʵ�����в�ͬ��ͨ��`getProxyClass0(��)`����ʵ�֣�ʵ���е��ô�����Ļ��棬�жϴ������ڻ������Ƿ��Ѿ����ڣ�����ֱ�ӷ��أ������������`proxyClassCache`��`valueFactory`���Խ��ж�̬���ɣ�`valueFactory`��`apply`�����������`getProxyClass(��)`�����߼����ơ�  

### 4. ʹ�ó���
#### 4.1 J2EE Web ������ Spring �� AOP(����������) ����
���ã�Ŀ�꺯��֮����  
������ Dao �У�ÿ�����ݿ��������Ҫ�������񣬶����ڲ�����ʱ����Ҫ��עȨ�ޡ�һ��д������ Dao ��ÿ�������������Ӧ�߼�����ɴ������࣬��϶ȸߡ�  
ʹ�ö�̬����ǰα�������£�  
```java
Dao {
    insert() {
        �ж��Ƿ��б����Ȩ�ޣ�
        ��������
        ���룻
        �ύ����
    }
    
    delete() {
        �ж��Ƿ���ɾ����Ȩ�ޣ�
        ��������
        ɾ����
        �ύ����
    }
}
```
ʹ�ö�̬�����α�������£�
```java
// ʹ�ö�̬�������ÿ������ĺ�������ÿ������ֻ��Ҫ��ע�Լ����߼����У��ﵽ���ٴ��룬����ϵ�Ч��
invoke(Object proxy, Method method, Object[] args)
                    throws Throwable {
    �ж��Ƿ���Ȩ�ޣ�
    ��������
    Object ob = method.invoke(dao, args)��
    �ύ����
    return ob; 
}            
``` 

#### 4.2 ���� REST �� Android ������������ Retrofit  
���ã����������������  
һ�������ÿ�������������Ƕ���Ҫ����һ��`HttpURLConnection`����`HttpClient`�������󣬻����� [Volley](https://github.com/android-cn/android-open-project-analysis/tree/master/volley) һ�������ȴ������У�Retrofit ����̶ȼ�����Щ������ʾ���������£�  
```java
public interface GitHubService {
  @GET("/users/{user}/repos")
  List<Repo> listRepos(@Path("user") String user);
}

RestAdapter restAdapter = new RestAdapter.Builder()
    .setEndpoint("https://api.github.com")
    .build();

GitHubService service = restAdapter.create(GitHubService.class);
```
�Ժ�����ֻ��Ҫֱ�ӵ���  
```java
List<Repo> repos = service.listRepos("octocat");
```
���ɣ�`Retrofit`��ԭ����ǻ��ڶ�̬������ͬʱ�õ��� [ע��](https://github.com/android-cn/android-open-project-analysis/blob/master/tech/annotation.md) ��ԭ�����Ĳ���������ܣ�������ȴ� [Retrofit  ʵ��ԭ�����](https://github.com/android-cn/android-open-project-analysis/tree/master/retrofit) ��ɡ�  