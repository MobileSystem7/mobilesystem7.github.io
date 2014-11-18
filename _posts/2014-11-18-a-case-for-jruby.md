---
layout: post
title: A Case for JRuby in the Enterprise
author: Matt Dew
tags: code jruby software java ruby interlock enterprise
category: Code
---

When we began developing [Interlock](http://www.mobilesystem7.com/interlock/) at [Mobile System 7](http://www.mobilesystem7.com), we, 
like all software teams, had to make difficult choices about our technology stack.  As a new startup 
developing a product that initially had abstract goals and no firm roadmap, not only did we not 
know what Interlock was going to become, but we [knew that we didn't know](http://youtu.be/GiPe1OiKQuk) 
everything about the customer environments into which Interlock might be deployed.

Our technology choices had to make sense in light of so many factors of relatively equal importance
(product goals, team skill set, requirements, time, money, customer infrastructure, on and on..) that it 
was **almost** as challenging to settle on something as it has been to design and build some of 
the more critical features in Interlock itself.  Yet we had to choose something.

For Interlock, our primary challenge was to choose a technology stack
that would help us maximize developer productivity and enable iteration over product features that 
were constantly evolving, while not leaving us unable to meet the understandably high 
technical standards and expectations of our enterprise customers.  

That's where [JRuby](http://jruby.org/) came in.  For those who aren't aware, JRuby is a Java implementation 
of the Ruby programming language - i.e. Ruby on the JVM.  JRuby development is led by a [small team](https://twitter.com/jruby) 
currently at Red Hat, but has seen significant community participation and in the last few years 
has graduated from being a curiosity in the Ruby community to a well-respected, first-class Ruby 
implementation - enjoying wide adoption and support.  The fact that Rubyists are [willing to be in 
the same room](http://rubyconf.org/program#prop_747) as the word Java (or JAnything) is as good a testament to JRuby's value as anything - 
because there was a time when the Java and Ruby communities were as philosophically opposed as 
[neckbeards](https://twitter.com/neckbeardhacker) and [hipsters](https://twitter.com/hipsterhacker).  

Despite progress in the community, a philosophical division seems to linger with respect to when and where 
it's appropriate to use Ruby over Java.  Ruby doesn't often come to mind when you mention enterprise 
software (particularly not off-the-shelf software), whereas Java is one of a small number of languages that is
almost synonymous with it.  Similarly, Java doesn't often come to mind when you think lean/agile 
startup, whereas Ruby has been credited (right or wrong) with helping many startups get features built and 
validated by customers quickly.

I think JRuby and other non-Java JVM languages are going to shatter those divisions, if they haven't already.  

JRuby's technical details are far more interesting than I can describe in a short blog post, and [people 
more intelligent than me](https://twitter.com/headius) have [explained them better](http://en.wikipedia.org/wiki/JRuby) than I can, anyway.
You can also find tutorials and code examples on the [JRuby Wiki](https://github.com/jruby/jruby/wiki) if you're interested in
more detail about how to get started with JRuby.

I'm going to focus instead on **some** of the reasons we believe JRuby so perfectly meets Mobile System 7's needs, 
and on how we arrived at that conclusion.  It ultimately boiled down to a few things:

  * [We are developing an enterprise product](#enterprise)
  * [We need to be responsive to nearly constant change](#agile)
  * [We want our feature set and architecture to mature seamlessly, with minimal disruption to our product roadmap](#maturity)
  * [We need to be able to deploy amongst enterprise application servers and services](#deployment)

## <a name="enterprise">We're developing an enterprise product</a>

First and foremost, we knew we were building an enterprise product.  Enterprises are not typically
the wild west of software, and for good reason.  Enterprise IT shops
have thousands of users and dozens (if not hundreds) of applications to support, and their 
users expect critical applications and data to be available whenever and from wherever they need it.
Application availability/stability directly affects workforce productivity, which in turn affects the bottom line.

Also, enterprises have generally made enormous investments in infrastructure, and they expect software products
to be able to operate within that infrastructure as much as possible.  Servers, operating systems, databases, 
user directories (AD, LDAP, etc), and countless other technologies are standardized, centralized, and managed 
independently of the applications that use them - ultimately so IT teams can make more efficient use
of their technical and human resources.

Consequently, enterprise IT shops focus on finding efficiencies and managing risk 
as much as they focus on pushing the feature envelope for their users - and oftentimes the risk management 
side of the coin wins the toss.  

No matter how aggressively a product vendor wants to help an enterprise 
push the envelope, that vendor's products are going to be subject to existing technical constraints - and
it's in the enterprise's and the vendor's interests to make sure products operate within those constraints
as much as possible.  

### You want to install what?

So, we didn't want to get down the long enterprise sales road only to have the [Interlock](http://www.mobilesystem7.com/interlock/) technology stack be a 
non-starter (or even a minor concern) for IT departments.  We had to avoid the "you want to install what on 
our servers?" question, and in my mind that left us with precious few choices: 
the .NET framework or the JVM  (and maybe C if we had been brave, but we're not).  

We could have attempted to convince our customers to let us install some other language interpreter or runtime 
(and whatever dependencies came with it), but that's a far more difficult sell than
using something they're already comfortable with. 

The JVM was the best choice because it gave us and our customers flexibility over the environments in which
Interlock could be deployed.  Plus, most enterprises are comfortable with Java and already have
an application install base that requires it.  Asking a customer to install the JRE as a dependency
for your software is not usually a tough sell, because in all likelihood it's already in use.

Fortunately, JRuby applications run in the JVM!  

## <a name="agile">Java may be ubiquitous in the enterprise, but it's not agile...man....</a>

I love Java.  The world is full of great Java developers, enterprises have been
using it for years, and the performance and stability of the JVM just gets better
and better.  

Java also has a huge ecosystem.  There are countless active open source projects, so chances
are you can scour <a href="https://github.com/search?utf8=%E2%9C%93&q=java">Github</a> (or <a href="http://sourceforge.com">Sourceforge</a> if you're nasty) 
and find any number of Java projects that you can use or alter
free-of-charge to help get your project off the ground.  The open source community continues to amaze
me with the quality and quantity of the software it produces - particularly the Java
community.  

Java is fast.  There was a time when people considered it to be a bit of a dog, but 
that time has definitely passed.  All the big guys are using Java these days, and they're 
likely dealing with performance and stability requirements that dwarf those of
99% of enterprise products.  If Java is good enough for them, it's probably good
enough for us and our customers.  
 
### But?
 
When I think of using Java to build an enterprise product with an amorphous feature set, I
start getting the sweats.  Call it a premonition, call it a crazy vision, call it intuition, 
but something told me that for a product like [Interlock](http://www.mobilesystem7.com/interlock/), Java would call for 
more developers, time, and lines of code than we had in our budget.

I don't think that's necessarily true of all projects or even all startups, but use  of Java frameworks
and conventions often implies significant up-front design, a relatively well-defined domain model, 
and nearly constant second guessing - because the cost of change is relatively high in the
Java world (I would look for a citation but we all know it's true).  

In an environment where change is constant - such as during the early development of Interlock - 
you have to minimize those costs by leaving yourself as able to react to change as possible. 
While your programming language of choice only contributes in part to your ability to change, 
in my experience it can be a significant contributor.

Ruby, in contrast, is great for the change heavy software project.  You can argue all you want
about its performance and manageability over time relative to Java, but in my opinion the language really
lends itself to projects that require frequent iteration over features.  You could write a super fast widget in Java, 
but you should be able to write one much more quickly in Ruby.  If all you
know is that you need a widget and that someone is going to be refining and validating its features over time,
I think it's wise to make every effort to get the widget developed and in front of that person 
as quickly as possible, solicit feedback, incorporate the feedback, repeat, repeat, repeat. 

All things being possible regardless of the language - I would probably not 
choose to write a widget in Java under those circumstances, even though it may be the better option 
once the widget's features are more well-defined. However, if you're
building an enterprise product you can't necessarily write the widget in whatever language is most familiar to you,
because at some point it'll be part of the product - or you'll have to trash it and rewrite it in something else 
that's more product-worthy.

Fortunately with JRuby you don't have to choose between Ruby and Java - you get both!  Code 
written in one can invoke code written in the other, and while it might feel weird to
implement a Java interface in Ruby, it's totally easy...and in time [you won't even care if anyone notices you're doing it](http://youtu.be/pIgZ7gMze7A). 

    //Poor example of a Java interface
    package com.mobilesystem;
  
    public interface Developer
    {
      public void   code();
      public String excuse();
    }

...

    #Poor example of a Java Interface being implemented in a Ruby Class
    class MattDew
      include com.mobilesystem7.Developer
  
      def code
        Laptop.new.open
        raise com.mobilesystem7.CodingProductivityException.new(self.excuse)
      end
      
      def excuse
        ["Can't code, in a meeting", 
         "Can't code, don't have headphones",
         "Can't code, lunch...",     
         "Can't code, the build is broken
        ].sample
      end
  
    end

If you choose to write a widget in Ruby in order to expedite feedback and then decide later on that it needs to be rewritten in Java (in part or in total), 
JRuby helps make the transition easier.  The code that uses the `MattDew` class doesn't have to care whether the class is implemented
in Ruby or Java, whether the implementation changes, or whether other classes that implement `com.mobilesystem7.Developer` are written 
in Ruby, Java, or both. JRuby takes care of the making sure it all works together seamlessly.

## <a name="maturity">Seamless, gradual maturation of product features and architecture</a>

[Interlock](http://www.mobilesystem7.com/interlock/)'s development has seen the gradual progression of various factors - a solidification
over time.  This maturation was anticipated, and we wanted to make sure our technology 
stack would facilitate it. 

Firstly, with respect to feature set, Interlock started off as a very abstract thing.  We knew
we wanted to enable enterprises to deploy Identity Analytics and Adaptive Access Control capabilities
against their critical services and data, but we had yet to work out the many details around **how** 
to do it and around which capabilities to focus on first.  

Over time we've worked out many specifics and our product has stabilized.  Change
is less frequent, though still common, and Interlock's primary features have gone from
being abstract to more-or-less concrete.  Our ability to change, react to customer feedback, and to
**quickly** put out stable features and seek out customer validation has been critical in 
the solidification of the feature set.

Secondly, with respect to language use, our codebase has seen the steady introduction of more
Java code over time - particularly in areas where performance is of utmost importance.  On several
occasions we've developed the first pass (or several iterations) of a feature in Ruby, written 
unit tests, and shipped a release, only to come back later and reimplement some of the 
code in Java.  In many cases we haven't even needed to make significant changes to the unit tests, which
generally are written using Ruby testing libraries even when the code being tested is written
in Java (the excellent Andrew Semprebon will be sharing how we use Ruby to test Java in a future post).

There's great comfort in knowing that when you make the effort to write good Java code that you've 
already validated the feature and written working tests for it, and that there's a **reduced** likelihood 
that you'll have to come back and make significant changes again.  Remember, change in Java is costly ([citation here](https://www.youtube.com/watch?v=DD0A2plMSVA)).

Thirdly, with respect to architecture, Interlock has progressed from having a monolithic library of
shared code to more specialized services/components that do a small number of things well. 
We have components written purely in Java, components written purely in Ruby, and components that
were written in both, and as much as possible we want those components to share libraries - particularly 
since the components emerged from a single codebase and were initially sharing code.  Being able to 
make that transition without significant disruption or product delays has been critical for us.

JRuby has been central in helping us progress on all three fronts.  Writing features in Ruby has enabled 
quick iteration and customer validation, and JRuby has allowed us to optimize in Java and separate capabilities
into components where appropriate - ultimately helping us reduce the cost of change and make 
steady progress without scrapping entire components or backing ourselves into a
performance bottleneck.  

## <a name="deployment">We need to deploy amongst enterprise application servers and services</a>

A dirty secret of the Ruby community that I don't think gets enough attention is the fact that 
container-managed services are nonexistent in Ruby application servers.  If you compare one
Ruby application server to another, all you're really comparing is their responsiveness.  This is because they
all have one feature, and one feature only - the ability to serve up your Ruby application.

Consequently, if you want nice things like authentication, message queueing,
background jobs, easy configuration of external integration points, in-memory caching, etc, 
you'll likely have to bake those things into your Ruby application with a gem and/or deploy additional 
technologies that are external to the application and its runtime.  That's OK when you're managing your own infrastructure, 
but when you're shipping a product to a customer it greatly increases the deployment and maintenance burden - and, thus,
the likelihood they're going to push back and say, "you want to install what?"

Java application servers, in contrast, have had these capabilities baked in for years.  When you're writing a Java web application,
for example, you simply develop against standard APIs, enable the services
behind those APIs in your application server, and then deploy your app.  The application server
manages the lifecycle of the application and of the services on which it depends, and coordinates their availability 
across whatever N servers the combined deployment requires.

In addition to providing services that applications use directly, Java application servers provide
simple configuration hooks into additional enterprise integration points.  Does your application need
a connection pool for access to a corporate LDAP service?  This would be no problem in Java land, via JNDI, 
but this simple, common feature is not something you'll find in a pure Ruby application server.  If you
need to use pure Ruby to establish an SSL connection to that LDAP server, or, worse still, authenticate to 
Active Directory via NTLM, welcome to hell (or at least to a few hours of [facepalming](http://www.blurrent.com/article/22-intriguing-reasons-your-mom-is-facepalm)).

Furthermore, many enterprises have made heavy investments in Java application servers and 
expect products to be able to run in their application server of choice - or at least for you
to make a compelling case as to why yours shouldn't have to.  

Not only is it wise to be able to install within an enterprise's application server of choice, but if you
build an application with the expectation that it will use container-managed services, you avoid having
to concern yourself with figuring out which gems or external services to use - and, more importantly, how to 
manage their deployment, maintenance, and availability in your customers' environments.

Fortunately, JRuby enables you to run your Ruby application as a Java application.  You can deploy
within WebLogic, WebSphere, Tomcat, JBoss, Geronimo, etc, and make use of whatever
container-managed services they provide.  While the code can be almost 100% Ruby, to the application server 
it all ends up running as Java bytecode.  No system dependencies are needed other than that which the customer
already has in support of their existing Java applications - namely, the JRE and a Java application server.

Our ability to focus on product features instead of concerning ourselves with questions about
deployment and integration has been of great benefit to our product and our customers, and without
JRuby I believe it would've been a much more difficult road.  

The bottom line is - we're writing a Java application primarily in Ruby, and enjoying the benefits of both.  

*NOTE: The [TorqueBox](http://torquebox.org) project is something we really love, and is worth looking into...because they **get it**.  They're 
making the convenience and full-featuredness of Java application servers available to Ruby applications via straightforward Ruby
APIs and declarative configuration.  There's more to it than that, of course, but I like to say they're doing for Ruby application 
**deployment** what Rails did for web application **development** - I really think it's that important.*

## Conclusion

Like working in an enterprise IT department, choosing languages and frameworks for
an enterprise software product is an exercise in determining how to optimize productivity 
and minimize risk - and in understanding and accepting tradeoffs.  There's rarely a right answer, and every choice
has the potential to influence the speed with which your team develops; the speed with 
which you're able to change or respond to customer needs; the speed with which you can install, 
update, and support your product; and, of course, the speed with which your product runs.  

While the factors that went into our choices are certainly more complicated than can be explained
in a blog post (and this has already gone on too long), we were focused on enabling an easy and 
appropriate transition from flexibility to maturity in our product, and on doing it
in a way that allowed us to operate within the technical constraints of our customers.  These needs
are primarily what led us to JRuby. 

Two years into the development of [Interlock](http://www.mobilesystem7.com/interlock/), I'm confident that we made the right choice in 
the JVM and in JRuby - and with the [interesting work](https://github.com/jruby/jruby/wiki/Truffle) 
going on in the JRuby community I'm confident we'll be able to take advantage of future benefits that 
we haven't yet imagined.  

Your mileage will vary, of course, and I genuinely wouldn't expect otherwise. 
JRuby and the Java ecosystem have many great strengths, but there are certainly
circumstances under which they aren't the appropriate choice.  Hopefully this post has helped highlight at least
one case where JRuby may be a good choice for you, and if you're interested 
in discussing further please reach out to me on Twitter at [@matt_dew](http://twitter.com/matt_dew).




