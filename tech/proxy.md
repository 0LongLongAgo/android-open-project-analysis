###��̬����

#####һ��ʲô�Ƕ�̬����
- 1. ������⼸������: ί����ʹ����� ��ί����ָ���Ǳ������ࣩ  
- 2. ��̬����ָ����ͨ�����������ί�����һЩ������
- 3. ��̬��������ṩ����һ������ķ��ʣ�ͬʱ����ʵ�ʶ���ľ�����ʵ�� ��һ�����ί�����е�ĳЩ��������������Ҫ�ģ� ����ͨ��������ʵ���Լ���Ҫ��Ч����

#####������̬�����ʵ�֣�
```java
	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	import java.lang.reflect.Proxy;

	/**
	 * @author Caij
	 */
	public class ProxyDemo {
		
		
		public static void main(String[] args) {
			/**
			 * ����ֻ�ܴ���������ʵ�ֽӿڵķ����� ���Ա��������ʵ�ֽӿ�
			 * */
			final CurrentClass currentClass = new CurrentClass();
			//��������ʵ�ֵĽӿ�
			Class<?>[] interfaces = new Class[]{ProxyInterface.class};
			//�����ʵ�ʵ�ί�����������д
			interfaces = currentClass.getClass().getInterfaces();
			ProxyInterface proxy = (ProxyInterface) Proxy.newProxyInstance(currentClass.getClass().getClassLoader(), 
						interfaces , new Handler(currentClass));
			proxy.sayGood();
		}
	}

	class Handler implements InvocationHandler {
		
		private Object object;
		
		public Handler(Object ob) {
			this.object = ob;
		}
		
		public Handler() {
		}

		/* (non-Javadoc)
		 * @see java.lang.reflect.InvocationHandler#invoke(java.lang.Object, java.lang.reflect.Method, java.lang.Object[])
		 * ��������е��ýӿ��еķ���
		 */
		@Override
		public Object invoke(Object proxy, Method method, Object[] args)
				throws Throwable {
			if (method.getName().equals("sayGood")) {
				System.out.println("very good"); //��Ҫ����ķ����޸�
				return null; //  ����ֵΪ�� ֱ�ӷ���null
			}
			return object == null ? null : method.invoke(object, args); //����Ҫ�޸ĵķ�������CurrentClass�ķ���
		}
		
	}

	interface ProxyInterface {
		public void sayHello();
		public void sayGood();
	}

	class CurrentClass implements ProxyInterface {

		@Override
		public void sayHello() {
			System.out.println("hello");
		}

		@Override
		public void sayGood() {
			System.out.println("good");
		}
	}
```
#####������̬�����ʵ��ԭ��
- 1. �ṩί����ʵ�ֵĽӿ�
- 2. ͨ��ʵ�ֵĽӿ�����ʵ���࣬ Ȼ���ڹ��췽���д���InvocationHandler ����ÿ��ʵ�ַ����е���InvocationHandler��invoke()��
- 3. Ȼ����дinvoke() ������ ���Լ���Ҫ���߼����롣

```java
    /** 
     * loader:������� 
     * interfaces:Ŀ�����ʵ�ֵĽӿ� 
     * h:InvocationHandler��ʵ���� 
     */  
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
                // ���ô������Ĺ��췽����Ҳ����$Proxy0(InvocationHandler h)��  
            Constructor cons = cl.getConstructor(constructorParams);  
                // ���ɴ������ʵ������InvocationHandler��ʵ���������Ĺ��췽��  
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
		
		/** 
		* ���������ʵ����ͨ���ṩ�Ľӿڲ���һ��ʵ������ֽ���
		*/ 
		public static Class<?> getProxyClass(ClassLoader loader,   
                                             Class<?>... interfaces)  
        throws IllegalArgumentException  
        {  
        // ���Ŀ����ʵ�ֵĽӿ�������65535�����׳��쳣����XX��˭��д��ôNB�Ĵ��밡����  
        if (interfaces.length > 65535) {  
            throw new IllegalArgumentException("interface limit exceeded");  
        }  
      
        // ������������������Class�����е��ֿڣ�  
        Class proxyClass = null;  
      
        String[] interfaceNames = new String[interfaces.length];  
      
        Set interfaceSet = new HashSet();   // for detecting duplicates  
      
        // ����Ŀ������ʵ�ֵĽӿ�  
        for (int i = 0; i < interfaces.length; i++) {  
              
            // �õ�Ŀ����ʵ�ֵĽӿڵ�����  
            String interfaceName = interfaces[i].getName();  
            Class interfaceClass = null;  
            try {  
            // ����Ŀ����ʵ�ֵĽӿڵ��ڴ���  
            interfaceClass = Class.forName(interfaceName, false, loader);  
            } catch (ClassNotFoundException e) {  
            }  
            if (interfaceClass != interfaces[i]) {  
            throw new IllegalArgumentException(  
                interfaces[i] + " is not visible from class loader");  
            }  
      
            // �м�ʡ����һЩ�޹ؽ�Ҫ�Ĵ��� .......  
              
            // ��Ŀ����ʵ�ֵĽӿڴ����Class����ŵ�Set��  
            interfaceSet.add(interfaceClass);  
      
            interfaceNames[i] = interfaceName;  
        }  
      
        // ��Ŀ����ʵ�ֵĽӿ�������Ϊ���棨Map���е�key  
        Object key = Arrays.asList(interfaceNames);  
      
        Map cache;  
          
        synchronized (loaderToCache) {  
            // �ӻ����л�ȡcache  
            cache = (Map) loaderToCache.get(loader);  
            if (cache == null) {  
            // �����ȡ���������½��ظ�HashMapʵ��  
            cache = new HashMap();  
            // ��HashMapʵ���͵�ǰ�������ŵ�������  
            loaderToCache.put(loader, cache);  
            }  
      
        }  
      
        synchronized (cache) {  
      
            do {  
            // ���ݽӿڵ����ƴӻ����л�ȡ����  
            Object value = cache.get(key);  
            if (value instanceof Reference) {  
                proxyClass = (Class) ((Reference) value).get();  
            }  
            if (proxyClass != null) {  
                // �����������Classʵ���Ѿ����ڣ���ֱ�ӷ���  
                return proxyClass;  
            } else if (value == pendingGenerationMarker) {  
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
            // �м�ʡ����һЩ���� .......  
              
            // ������Ƕ�̬���ɴ���������ؼ��ĵط�  
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(  
                proxyName, interfaces);  
            try {  
                // ���ݴ�������ֽ������ɴ������ʵ��  
                proxyClass = defineClass0(loader, proxyName,  
                proxyClassFile, 0, proxyClassFile.length);  
            } catch (ClassFormatError e) {  
                throw new IllegalArgumentException(e.toString());  
            }  
            }  
            // add to set of all generated proxy classes, for isProxyClass  
            proxyClasses.put(proxyClass, null);  
      
        }   
        // �м�ʡ����һЩ���� .......  
          
        return proxyClass;  
        }  
```

#####�ġ���̬����װ�Σ� �̳еıȽϡ�
���룺 �����CurrentClass �� ProxyInterface ��һ�д���ʵ���еĶ���
```java
	/**
	 * �̳��޸ķ���
	 * @author Caij
	 */
	class ExtendsClass extends CurrentClass {

		/* (non-Javadoc)
		 * @see com.caij.proxy.CurrentClass#sayHello()
		 * ��Ϊ��������㲻���⣬ ��д
		 */
		@Override
		public void sayHello() {
			System.out.println("very good");
		}
	}

	/**
	 * @author Caij
	 * װ��  �޸ķ���
	 */
	class DecorateClass implements ProxyInterface {
		
		private CurrentClass clazz;
		
		public DecorateClass(CurrentClass currentClass) {
			this.clazz = currentClass;
		}

		/* (non-Javadoc)
		 * @see com.caij.proxy.ProxyInterface#sayHello()
		 * �ڲ���Ҫ�ı�ķ���ֱ�ӵ��ñ�װ����ķ���
		 */
		@Override
		public void sayHello() {
			clazz.sayGood();
		}

		/* (non-Javadoc)
		 * @see com.caij.proxy.ProxyInterface#sayGood()
		 * ����Ҫ�ı�ķ�����д�Լ���Ҫ���߼�
		 */
		@Override
		public void sayGood() {
			System.out.println("very good");
		}
	}
```

- 1.��ͬ�㣺  
	* �����Ըı�����еķ�����
- 2.��ͬ��
	* �̳к�װ�θ���ע�ص��ǶԹ��ܵļ�ǿ�� �����ǶԹ��ܵ�������
	* ʹ�ó�����ͬ�� ��û�о�������ʱ��װ����ʵ�ֲ��˵ġ� �̳л���ɴ����Ĵ������࣬ ������Ҫʵ�ֵ���һ���ӿڣ� �Ǹ��ӿ���20�������� �ͱ�����д20�������������ֻ�����ʹ�ö�̬����

#####�塢����ʹ�ó���
- 1. android�м����¼��Ĵ���  
	* ���ɣ� �����ӿ���û��ʵ���ģ� ʹ�ü̳л���װ�εĻ���Ҫʵ�ֽӿڣ� �����¼������ʱ����ͨ�����似���� ���Ҫ����ʵ�ּ����ӿڵĶ�����ɴ������࣬ ��չ�Բ