---
layout: post
title: Imposterizing checked exceptions
category: code
---

I have been frustrated many times by some of Java's checked exceptions. I'm sure you can recall some of your own experiences where you wished a `CheckedException` was a `RuntimeException` instead.

Let me share a useful exception idiom that I've been using for a long time and on lots of projects, which I call the `ExceptionImposter`.

We all know the Java standard libraries use checked exceptions to signal problems that can only occur at runtime either if we have screwed up or if something really bad has happened - something we cannot do anything about anyway. Personally, since I use Java reflection capabilities a lot, I have to deal with the `InstantiationException` and the `IllegalAccessException` too often for my taste.

For example, whenever I want to dynamically instanciate a class, even if I know the code should never fail, the underlying Java code throws a checked exception and I end up with code that looks like this:

{% highlight java %}
        ...
        try {
            return WebDriverFactory.class.cast(webDriverFactoryClass.newInstance());
        } catch (Exception shouldNeverOccur) {
            // now what?
        }    
        ...
{% endhighlight %}

If I know that the `webDriverFactoryClass` has a public no-arg constructor that does nothing, the code cannot fail. If it fails, it could be that something is wrong with my environment or that I made an error while programming the class. That's something I will catch during my development cycle but that I don't expect to occur afterwards. 
I cannot just ignore the exception, because if something really went wrong, the application will not behave correctly and I will have a hard time diagnosing the failure and finding the source of the error.

I know I must throw another exception and I want that exception to be a runtime exception because there's nothing useful the calling code can do about the error. I can wrap the exception into some generic `RuntimeException` but that's not very satisfying. It will not help me find the error and it will add another level of nesting and make the stack trace harder to read. I find we already have too many levels of nested exceptions which make framework exceptions very hard to read.

I could also write my own wrapper to rethrow a checked exception. It think it would be slightly better, since I can now name the error, but what still annoys me is that other level of nesting and the longer stack trace.

Instead, I use the `ExceptionImposter` to mimic the checked exception and turn it into a runtime exception:

{% highlight java %}
    ...
    try {
        return WebDriverFactory.class.cast(webDriverFactoryClass.newInstance());
    } catch (Exception shouldNeverOccur) {
        ExceptionImposter.imposterize(shouldNeverOccur);
    }    
    ...
{% endhighlight %}

Here's how it works:

{% highlight java %}
    public class ExceptionImposter extends RuntimeException {
        private final Exception imposterized;
      
        public static RuntimeException imposterize(Exception e) {
            if (e instanceof RuntimeException) return (RuntimeException) e;
        
            return new ExceptionImposter(e);
        }
      
        public ExceptionImposter(Exception e) {
            super(e.getMessage(), e.getCause());
            imposterized = e;
            setStackTrace(e.getStackTrace());
        }
      
        public Exception getRealException() {
            return imposterized;
        }
      
        public String toString() {
            return imposterized.toString();
        }
    }
{% endhighlight %}    

As you can see from the following test case, the `ExceptionImposter` looks like the real exception as much as possible:

{% highlight java %}
    public class ExceptionImposterTest {
        private Exception realException;
    
        @Test
        public void leavesUncheckedExceptionsUnchanged() {
            realException = new RuntimeException();
            assertSame(realException, ExceptionImposter.imposterize(realException));
        }
    
        @Test
        public void imposterizesCheckedExceptionsAndKeepsAReference() {
            realException = new Exception();
            RuntimeException imposter = ExceptionImposter.imposterize(realException);
            assertTrue(imposter instanceof ExceptionImposter);
            assertSame(realException, ((ExceptionImposter) imposter).getRealException());
        }
    
        @Test
        public void mimicsImposterizedExceptionToStringOutput() {
            realException = new Exception("Detail message");
            RuntimeException imposter = ExceptionImposter.imposterize(realException);
            assertEquals(realException.toString(), imposter.toString());
        }
    
        @Test
        public void copiesImposterizedExceptionStackTrace() {
            realException = new Exception("Detail message");
            realException.fillInStackTrace();
            RuntimeException imposter = ExceptionImposter.imposterize(realException);
            assertArrayEquals(realException.getStackTrace(), imposter.getStackTrace());
        }
    
        @Test
        public void mimicsImposterizedExceptionStackTraceOutput() {
            realException = new Exception("Detail message");
            realException.fillInStackTrace();
            RuntimeException imposter = ExceptionImposter.imposterize(realException);
            assertEquals(stackTraceOf(realException), stackTraceOf(imposter));
        }
    
        private String stackTraceOf(Exception exception) {
            StringWriter capture = new StringWriter();
            exception.printStackTrace(new PrintWriter(capture));
            capture.flush();
            return capture.toString();
        }
    }
{% endhighlight %}