Full article here: https://medium.com/androidiots/writing-your-own-annotation-processors-in-android-1fa0cd96ef11

# What are Annotations ?
An annotation is a form of syntactic <b>metadata</b> that can be added to Java source code.<br>
We can annotate classes, interfaces, methods, variables, parameters etc.<br>
Java annotations can be read from source files. Java annotations can also be embedded in and read from class files generated by the compiler.<br>
Annotations can be retained by Java VM at <i>run-time</i> and read via <i>reflection</i>.

```
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.FIELD)
public @interface BindView {
    int value();
}
```

Creating an annotation requires two pieces of information: [Retention](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Retention.html) and [Target](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Target.html)

A [RetentionPolicy](https://docs.oracle.com/javase/8/docs/api/index.html?java/lang/annotation/RetentionPolicy.html) specifies how long, in terms of the program lifecycle, the annotation should be retained for. For example, annotations may be retained during compile-time or runtime, depending on the retention policy associated with the annotation.

![](https://github.com/hitanshu-dhawan/AnnotationProcessing/blob/master/images/RetentionPolicy.png)

The Target of an annotation specifies which Java [ElementType](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/ElementType.html)s an annotation can be applied to.

![](https://github.com/hitanshu-dhawan/AnnotationProcessing/blob/master/images/ElementType.png)

# Why Annotation Processors ?
#### 1. <del>Runtime</del> Compile time
#### 2. No Reflection
#### 3. Generate boilerplate code*
\*Annotation processing can only be used to generate new files and not to modify existing ones

## How does Annotation Processing work ?
The annotation processing takes place in many rounds. The compiler reads a java source file with the registered annotations and calls their corresponding annotation processors which will generate more java source files with more annotations. These new annotations will again call their corresponding annotation processors which will again generate more java source files.
This cycle continues until no new java source file is generated in the cycle.

![](https://github.com/hitanshu-dhawan/AnnotationProcessing/blob/master/images/AnnotationProcessingRounds.png)

## How to register a Processor ?
A processor must be registered to the compiler so that it can run while the application is being compiled.

Annotation Processors can be registered in two ways.

#### 1.
Create a directory structure like this<br>
```<your-annotation-processor-module>/src/main/resources/META-INF/services```<br>
Now, in the services directory, we will create a file named ```javax.annotation.processing.Processor```.<br>
This file will list the classes (fully qualified name) that the compiler will call when it compiles the application's source code while annotation processing.

#### 2.
Use Google's [AutoService](https://github.com/google/auto/tree/master/service) library.<br>
Just annotate your Processor with ```@AutoService(Processor.class)```<br>

Example:
```
package foo.bar;

import javax.annotation.processing.Processor;

@AutoService(Processor.class)
final class MyProcessor implements Processor {
  // …
}
```

## How to create a Processor ?
To create our custom Annotation Processor we need to make a class that extends [AbstractProcessor](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/AbstractProcessor.html) which defines the base methods for the processing.<br>
We have to override four methods to provide our implementations for the processing.

```
public class Processor extends AbstractProcessor {

    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
        // initialize helper/utility classes...
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnvironment) {
        // do processing...
        return true;
    }

    @Override
    public Set<String> getSupportedAnnotationTypes() {
        return new HashSet<String>() {{
            add(BindView.class.getCanonicalName());
            add(OnClick.class.getCanonicalName());
        }};
    }

    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }
}
```

<b>```init()```</b>
gives you Helper/Utility classes like [Filer](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/Filer.html) (to generate files), [Messager](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/Messager.html) (for logging errors, warnings etc.), [Elements](https://docs.oracle.com/javase/8/docs/api/javax/lang/model/util/Elements.html) (utility methods for operating on program elements), [Types](https://docs.oracle.com/javase/8/docs/api/javax/lang/model/util/Types.html) (utility methods for operating on types) etc.<br>
You can get these classes with ```processingEnvironment```.

<b>```process()```</b>
is the method where all the processing happens.<br>
It gives you ```annotations``` that need to be processed and ```roundEnvironment``` provides the information about the round and has some utility methods for querying elements.<br>
e.g. ```processingOver()```,```getRootElements()```,```getElementsAnnotatedWith()``` etc.

<b>```getSupportedAnnotationTypes()```</b>
returns the names of the annotation types supported by this processor.

<b>```getSupportedSourceVersion()```</b>
returns the latest source version supported by this processor.

<b>Note</b> : You can also use ```@SupportedAnnotationTypes``` and ```@SupportedSourceVersion``` instead of ```getSupportedAnnotationTypes()``` and ```getSupportedSourceVersion()``` respectively.

# Demo #1 : Singleton
1. [Singleton.java](singleton-annotations/src/main/java/com/hitanshudhawan/singleton_annotations/Singleton.java)
2. [Processor.java](singleton-compiler/src/main/java/com/hitanshudhawan/singleton_compiler/Processor.java)

# Demo #2 : KSingleton
1. [KSingleton.kt](ksingleton-annotations/src/main/java/com/hitanshudhawan/ksingleton_annotations/KSingleton.kt)
2. [Processor.kt](ksingleton-compiler/src/main/java/com/hitanshudhawan/ksingleton_compiler/Processor.kt)

# Important Classes/Objects
<b>```ProcessingEnvironment```</b> : An annotation processing tool framework will provide an annotation processor with an object implementing this interface so the processor can use facilities provided by the framework to write new files, report error messages, and find other utilities.

<b>```Elements```</b> : Utility methods for operating on program elements. Can be accessed by ```ProcessingEnvironment.getElementUtils()```.

<b>```Types```</b> : Utility methods for operating on types. Can be accessed by ```ProcessingEnvironment.getTypeUtils()```.

<b>```Messager```</b> : A Messager provides the way for an annotation processor to report error messages, warnings, and other notices. Can be accessed by ```ProcessingEnvironment.getMessager()```.

<b>```Filer```</b> : This interface supports the creation of new files by an annotation processor. Can be accessed by ```ProcessingEnvironment.getFiler()```.

<br>

<b>```RoundEnvironment```</b> : An annotation processing tool framework will provide an annotation processor with an object implementing this interface so that the processor can query for information about a round of annotation processing. We can get our desired elements with ```RoundEnvironment.getRootElements()``` and ```RoundEnvironment.getElementsAnnotatedWith()``` methods.

<br>

<b>```ElementFilter```</b> : Filters for selecting just the elements of interest from a collection of elements. Contains methods like ```ElementFilter.constructorsIn()```, ```ElementFilter.methodsIn()```, ```ElementFilter.fieldsIn()``` etc.

# How to generate .java files ?
We can use Square's [JavaPoet](https://github.com/square/javapoet) library for generating ```.java``` files.<br>
JavaPoet makes it really simple to define a class structure and write it while processing. It creates classes that are very close to a handwritten code.

### Example
This ```HelloWorld``` class
```
package com.example.helloworld;

public final class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello, JavaPoet!");
  }
}
```
can be generated by this piece of code
```
MethodSpec main = MethodSpec.methodBuilder("main")
    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
    .returns(void.class)
    .addParameter(String[].class, "args")
    .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(main)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);
```

<b>Note</b> : There's also another Square library [KotlinPoet](https://github.com/square/kotlinpoet) for generating ```.kt``` files.

# Demo #3 : ButterKnife
#### Annotations
1. [BindView.java](butterknife-annotations/src/main/java/com/hitanshudhawan/butterknife_annotations/BindView.java)
2. [OnClick.java](butterknife-annotations/src/main/java/com/hitanshudhawan/butterknife_annotations/OnClick.java)
#### Processor
3. [Processor.java](butterknife-compiler/src/main/java/com/hitanshudhawan/butterknife_compiler/Processor.java)
#### Library
4. [ButterKnife.java](butterknife/src/main/java/com/hitanshudhawan/butterknife/ButterKnife.java)

### Example
This ButterKnife library will generate separate classes (with suffix ```Binder```) for each of the Activity where we used the annotations ```@BindView``` or ```@OnClick```.<br>

<b>```MainActivity.java```</b>
```
public class MainActivity extends AppCompatActivity {

    private int numberOfTimesTextViewClicked = 0;

    @BindView(R.id.text_view)
    TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ButterKnife.bind(this);
    }

    @OnClick(R.id.text_view)
    void onTextViewClicked(View view) {
        textView.setText(String.valueOf(++numberOfTimesTextViewClicked));
    }
}
```
<b>```MainActivityBinder.java```</b>
```
public class MainActivityBinder {
    public MainActivityBinder(MainActivity activity) {
        bindViews(activity);
        bindOnClicks(activity);
    }

    private void bindViews(MainActivity activity) {
        activity.textView = (TextView) activity.findViewById(2131165314);
    }

    private void bindOnClicks(final MainActivity activity) {
        activity.findViewById(2131165314).setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                activity.onTextViewClicked(view);
            }
        });
    }
}
```

# References
<ul>
    <li>https://github.com/MindorksOpenSource/annotation-processing-example</li>
    <li>https://youtu.be/2Dlo8OSwzaI</li>
    <li>https://youtu.be/cBD4aiJ8EDo</li>
    <li>https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/package-summary.html</li>
    <li>https://docs.oracle.com/javase/7/docs/jdk/api/apt/mirror/overview-summary.html</li>
    <li>https://kotlinlang.org/docs/reference/annotations.html</li>
    <li>https://stackoverflow.com/q/2123118</li>
    <li>https://docs.oracle.com/javase/7/docs/technotes/tools/solaris/javac.html#processing</li>
    <li>https://docs.oracle.com/javase/1.5.0/docs/guide/apt/GettingStarted.html</li>
    <li>https://docs.oracle.com/javase/7/docs/technotes/guides/apt/index.html</li>
    <li>https://kotlinlang.org/docs/reference/kapt.html</li>
    <li>https://docs.oracle.com/javase/8/docs/api/javax/lang/model/package-summary.html</li>
    <li>https://bracha.org/mirrors.pdf</li>
    <li>https://docs.oracle.com/javase/7/docs/jdk/api/apt/mirror/overview-summary.html</li>
    <li>http://hannesdorfmann.com/annotation-processing/annotationprocessing101</li>
</ul>
