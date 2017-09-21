### Java 类分析器

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.util.Scanner;

public class ClassAnalyzer {
    private static final String tab = "    ";//缩进
   
    public static void analyzer(String className) throws ClassNotFoundException{
        Class c = Class.forName(className);
        System.out.print(Modifier.toString(c.getModifiers()));
        System.out.print(" ");
        System.out.print(c.toString());
        Class superC = c.getSuperclass();
        if(superC != null){
            System.out.print(" extends "+superC.getName());
        }
        System.out.println("{");//类开始括号

        //打印域
        System.out.println(tab+"//域");
        Field[] fields = c.getDeclaredFields();
        for(Field field:fields){
            printField(field);
        }

        //打印构造器
        System.out.println(tab+"//构造器");
        Constructor[] constructors = c.getDeclaredConstructors();
        for(Constructor constructor:constructors){
            printConstructor(constructor);
        }

        //打印方法
        System.out.println(tab+"//方法");
        Method[] methods = c.getDeclaredMethods();
        for(Method method:methods){
            printMethod(method);
        }
        System.out.println("}");//类结束括号
    }

    //打印域
    private static void printField(Field field){
        System.out.print(tab);
        System.out.print(Modifier.toString(field.getModifiers()));
        System.out.print(" ");
        Class fieldType = field.getType();
        if(fieldType.isArray()){
            System.out.print(getArrayTypeName(fieldType));
        }else{
            System.out.print(field.getType().getName());
        }
        System.out.print(" ");
        System.out.print(field.getName());
        System.out.println(";");
    }

    //打印构造器
    private static void printConstructor(Constructor constructor){
        System.out.print(tab);
        System.out.print(Modifier.toString(constructor.getModifiers()));
        System.out.print(" ");
        System.out.print(constructor.getDeclaringClass().getSimpleName());
        Class[] varTypes = constructor.getParameterTypes();
        System.out.print("(");
        printParameters(varTypes);
        System.out.println(");");
    }

    //打印方法
    private static void printMethod(Method method){
        System.out.print(tab);
        System.out.print(Modifier.toString(method.getModifiers()));
        System.out.print(" ");
        Class returnType = method.getReturnType();
        if(returnType.isArray()){
            System.out.print(getArrayTypeName(returnType));
        }else{
            System.out.print(method.getReturnType().getName());
        }
        System.out.print(" ");
        System.out.print(method.getName());
        System.out.print("(");
        Class[] varTypes = method.getParameterTypes();
        printParameters(varTypes);
        System.out.print(")");
        //声明抛出的异常
        Class[] exceptionType = method.getExceptionTypes();
        if(exceptionType.length != 0){
            System.out.print(" throws ");
            for(int i=0;i<exceptionType.length;i++){
                System.out.print(exceptionType[i].getName());
                if(i < (exceptionType.length - 1)){
                    System.out.print(",");
                }
            }
        }
        System.out.println(";");
    }

    //打印构造器和方法的参数列表
    private static void printParameters(Class[] varTypes){
        if(varTypes.length > 0){
            for(int i = 0; i < varTypes.length; i++){
                if(varTypes[i].isArray()){
                    System.out.print(getArrayTypeName(varTypes[i]));
                }else{
                    System.out.print(varTypes[i].getName());
                }
                if( i < (varTypes.length - 1)){
                    System.out.print(", ");
                }
            }
        }else{
            System.out.print(" ");
        }
    }

   
    public static String getArrayTypeName(Class type){
        StringBuffer buffer = new StringBuffer(getArrayType(type).getName());
        int dimension = countArrayDimension(type);
        for(int i=1;i<=dimension;i++){
            buffer.append("[]");
        }
        return buffer.toString();
    }

   
    public static int countArrayDimension(Class type){
        int dimension = 0;
        if(type.isArray()){
            Class tempType = type;
            while((tempType = tempType.getComponentType()) != null){
                dimension++;
            }
        }
        return dimension;
    }

   
    public static Class getArrayType(Class type){
        Class arrayType = null;
        if(type.isArray()){
            Class tempType = type.getComponentType();
            do{
                arrayType = tempType;
            }while((tempType = tempType.getComponentType()) != null);
        }
        return arrayType;
    }
   
    public static void main(String[] args) {
        try {
            Scanner in = new Scanner(System.in);
            System.out.print("Input class name:");
            String className = in.next();
            in.close();
            analyzer(className);
        } catch (ClassNotFoundException ex) {
            ex.printStackTrace();
        }
    }
}import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.util.Scanner;

public class ClassAnalyzer {
    private static final String tab = "    ";//缩进
   
    public static void analyzer(String className) throws ClassNotFoundException{
        Class c = Class.forName(className);
        System.out.print(Modifier.toString(c.getModifiers()));
        System.out.print(" ");
        System.out.print(c.toString());
        Class superC = c.getSuperclass();
        if(superC != null){
            System.out.print(" extends "+superC.getName());
        }
        System.out.println("{");//类开始括号

        //打印域
        System.out.println(tab+"//域");
        Field[] fields = c.getDeclaredFields();
        for(Field field:fields){
            printField(field);
        }

        //打印构造器
        System.out.println(tab+"//构造器");
        Constructor[] constructors = c.getDeclaredConstructors();
        for(Constructor constructor:constructors){
            printConstructor(constructor);
        }

        //打印方法
        System.out.println(tab+"//方法");
        Method[] methods = c.getDeclaredMethods();
        for(Method method:methods){
            printMethod(method);
        }
        System.out.println("}");//类结束括号
    }

    //打印域
    private static void printField(Field field){
        System.out.print(tab);
        System.out.print(Modifier.toString(field.getModifiers()));
        System.out.print(" ");
        Class fieldType = field.getType();
        if(fieldType.isArray()){
            System.out.print(getArrayTypeName(fieldType));
        }else{
            System.out.print(field.getType().getName());
        }
        System.out.print(" ");
        System.out.print(field.getName());
        System.out.println(";");
    }

    //打印构造器
    private static void printConstructor(Constructor constructor){
        System.out.print(tab);
        System.out.print(Modifier.toString(constructor.getModifiers()));
        System.out.print(" ");
        System.out.print(constructor.getDeclaringClass().getSimpleName());
        Class[] varTypes = constructor.getParameterTypes();
        System.out.print("(");
        printParameters(varTypes);
        System.out.println(");");
    }

    //打印方法
    private static void printMethod(Method method){
        System.out.print(tab);
        System.out.print(Modifier.toString(method.getModifiers()));
        System.out.print(" ");
        Class returnType = method.getReturnType();
        if(returnType.isArray()){
            System.out.print(getArrayTypeName(returnType));
        }else{
            System.out.print(method.getReturnType().getName());
        }
        System.out.print(" ");
        System.out.print(method.getName());
        System.out.print("(");
        Class[] varTypes = method.getParameterTypes();
        printParameters(varTypes);
        System.out.print(")");
        //声明抛出的异常
        Class[] exceptionType = method.getExceptionTypes();
        if(exceptionType.length != 0){
            System.out.print(" throws ");
            for(int i=0;i<exceptionType.length;i++){
                System.out.print(exceptionType[i].getName());
                if(i < (exceptionType.length - 1)){
                    System.out.print(",");
                }
            }
        }
        System.out.println(";");
    }

    //打印构造器和方法的参数列表
    private static void printParameters(Class[] varTypes){
        if(varTypes.length > 0){
            for(int i = 0; i < varTypes.length; i++){
                if(varTypes[i].isArray()){
                    System.out.print(getArrayTypeName(varTypes[i]));
                }else{
                    System.out.print(varTypes[i].getName());
                }
                if( i < (varTypes.length - 1)){
                    System.out.print(", ");
                }
            }
        }else{
            System.out.print(" ");
        }
    }

   
    public static String getArrayTypeName(Class type){
        StringBuffer buffer = new StringBuffer(getArrayType(type).getName());
        int dimension = countArrayDimension(type);
        for(int i=1;i<=dimension;i++){
            buffer.append("[]");
        }
        return buffer.toString();
    }

   
    public static int countArrayDimension(Class type){
        int dimension = 0;
        if(type.isArray()){
            Class tempType = type;
            while((tempType = tempType.getComponentType()) != null){
                dimension++;
            }
        }
        return dimension;
    }

   
    public static Class getArrayType(Class type){
        Class arrayType = null;
        if(type.isArray()){
            Class tempType = type.getComponentType();
            do{
                arrayType = tempType;
            }while((tempType = tempType.getComponentType()) != null);
        }
        return arrayType;
    }
   
    public static void main(String[] args) {
        try {
            Scanner in = new Scanner(System.in);
            System.out.print("Input class name:");
            String className = in.next();
            in.close();
            analyzer(className);
        } catch (ClassNotFoundException ex) {
            ex.printStackTrace();
        }
    }
}
```

