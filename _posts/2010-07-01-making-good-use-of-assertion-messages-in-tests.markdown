---
layout: post
title: Making good use of assertion messages in tests
category: Code
---

Have you ever struggle with coming up with useful assertion messages in your tests? Well, I have; until not so long ago.

I remember when I started using JUnit. It was summer 2000, and I became addicted to writing tests first. I was already writing automated tests at that time, inspired by my lecture of [Thinking In Java](http://www.mindview.net/Books/TIJ/). Not using any automated unit test framework, I was writing my tests last in the good old _main()_ function. I had to inspect the outcome of my tests for correctness each time I run them, and that was painful.

JUnit was a discovery that changed the way I programmed and approached software development forever. Central to xUnit frameworks is the notion of making the tests self-checking by using __assertion methods__, basically utility methods that evaluate whether an expected outcome has been achieved. Now I had a way to express the expected outcome, let the computer check it for me, and produce a useful message for me (and others) - the human readers - to help diagnose problems. One of the goals of automating tests is to use tests as documentation. In case of test failure, what you want is for the test to act as a tracer bullet, helping you understand very quickly what the problem is. Assertion messages play a role in this. A well-crafted assertion message makes it clear which assertion fails and what the symptoms of the problem are. Now that's easy to say, the hard part is to figure out what the message should say.

As Nat Pryce and Steve Freeman explain in their [book](http://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627 "Growing Object-Oriented Software, Guided by Tests"), during the test-driven cycle, after you make the test fail, take a moment to read the assertion message, ask yourself what the reader will get out of it, and adjust to make the diagnostics clear. They provide a number of best practices in the book to help with test diagnostics. One is about making the assertion messages explanatory.

I have long been in the school of thought that strives to have a single assertion per test method and as result, for a long time I felt no need to use assertion messages. If you use small and focused tests and name them well, the tests will tell you most of what you need to know to diagnose the problem. For example, when the following test fails:

{% highlight java %}
calculatesGrandTotal() {
    String[] prices = { "50", "75.50", "12.75" };
    BigDecimal expectedTotal = new BigDecimal("138.25");

    for (String price : prices) {
        cart.add(anItem().priced(price).build());
    }
    assertThat(cart.getGrandTotal(), equalTo(expectedTotal));
}
{% endhighlight %}

the failure report can be considered enough to understand what the problem is:

{% highlight java %}
Expected: <138.25>
     but: was <139.25>
	at org.hamcrest.MatcherAssert.assertThat(MatcherAssert.java:20)
	at org.hamcrest.MatcherAssert.assertThat(MatcherAssert.java:8)
	at test.com.pyxis.petstore.domain.order.CartTest.calculatesGrandTotal(CartTest.java:53)
	...
{% endhighlight %}

Indeed, the name of the test tells us that the cart fails to calculate its grand total correctly. The assertion messages shows the difference between the expected and actual outcome. Yet we can make the diagnostics even easier for the reader by simply adding an assertion message to identify the value being asserted:

{% highlight java %}
    assertThat("grand total", cart.getGrandTotal(), equalTo(expectedTotal));
{% endhighlight %}

See how that helps:

{% highlight java %}
grand total 
Expected: <138.25>
     but: was <139.25>
{% endhighlight %}

I've come to adopt that practice since reading Pryce and Freeman's book. Assertion messages are not used as often as they should. They can really help make failure reports more helpful, whether you have a single or multiple assertions in your test.

Another help in making assertion reports clearer comes from using [Hamcrest matchers](http://code.google.com/p/hamcrest/wiki/Tutorial) and _assertThat()_. Consider the following example:

{% highlight java %}
findsItemsByNumber() throws Exception {
    havingPersisted(product);
    havingPersisted(anItem().of(product).withNumber("12345678"));

    Item found = itemInventory.find(new ItemNumber("12345678"));
    assertThat("available inventory", found, itemWithNumber("12345678"));
}
{% endhighlight %}

In case of failure, here is what we get:

{% highlight java %}
java.lang.AssertionError: available inventory
Expected: an item with number "12345678"
     but: number was "87654321"
	at org.hamcrest.MatcherAssert.assertThat(MatcherAssert.java:20)
	at org.hamcrest.MatcherAssert.assertThat(MatcherAssert.java:8)
	at test.integration.com.pyxis.petstore.persistence.PersistentItemInventoryTest.findsItemsByNumber(PersistentItemInventoryTest.java:56)
{% endhighlight %}

which I hope make the point.
	
As Pryce and Freeman say, diagnostics are a first-class feature. I now do my best to make assertion messages helpful so that whoever has to change the code in the future will understand what the expected behavior is.




