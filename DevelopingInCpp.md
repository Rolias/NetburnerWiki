
Developing in Cpp

Moving from developing under C, or developing "pure" C++ in a desktop environment to doing Embedded C++ for the NetBurner can pose some unique challenges. Among other items you have to consider dynamic memory management and interoperability with C. 

==Getting Started With C++==
The easiest way to get started is to just write C code but put it in a filename that ends with .cpp and get the benefits of the C++ compiler. C++ has some stricter type checking rules that can improve your code. Once you are comfortable with the compiler you can start taking advantage of namespaces to isolate your code. Then you can start using static classes to take advantage of encapsulation, polymorphism and inheritance. While the general trend is to eschew static classes (among other things they make unit testing difficult) it does allow for minimizing dynamic memory and easy interoperability with C and the RTOS. Static classes can get you into trouble so don't do anything too fancy in their constructors and you'll probably at least want to read the section below on Initializing statics. Keep in mind the old aphorism about the coupling of great power and great responsibility. C++ gives you great power for for writing [http://en.wikipedia.org/wiki/Don't_repeat_yourself DRY] and [http://en.wikipedia.org/wiki/Solid_(Object_Oriented_Design) SOLID] code but you do have to take responsibility for learning to use it well. It's well worth the effort.

==Easy First Steps==
Start using the C++ standard libraries instead of the C libraries. Also be aware there is an important distinction between the C++ Standard TEMPLATE libraries, and the C++ standard Libraries. I also feel the STL is well worth using, but it does involve a steeper learning curve and is easy to use wrong, so using the STL isn't an easy first step.
So what is an easy first step. Start with the <iostream> and <sstream> libraries. You may need to add include paths to your settings to use these libraries. See the section below on the STL for how to add the paths if you've never done this. Remember all the C++ standard libraries are in the std namespace so up near your #includes you need to add either a ''using namespace std;'' line or my preference is to only specify the pieces I'm using. Here's an example:
<code style="font-size:medium;">
 using std::cout;
 using std::end; 
</code>
Alternately, you can just preface every use of an object from the std library with std:: but that get's really old really fast. I've never had a conflict with any name in the std:: namespace but I use naming conventions that avoid it. If you tend to use names like 'string' for your parameters or variables your mileage may vary. 

=== The printf is dead, long live cout ===
The <iostream> library contains that famous ''cout'' operator you read about when you first learned C++. The syntax seemed strange so you immediately went back to ''printf'' and its brethern but you shouldn't have. If you've ever trapped the NetBurner because you changed an int to a float (or vice versa) and forgot to change that little %d in a printf or sprintf, you already know why you should use cout. It's typesafe and printf isn't. In other words in C++ you wouldn't have trapped when you changed that variable from an int to a float. Heck, you could have changed it to a string or even a class of your own and you wouldn't have trapped. The only downside is the iostream library loads in lots of C++ functionality and therefore has a pretty heavy memory/flash footprint. See the [http://syncor.blogspot.com/2011/03/cost-of-type-safe-code.html blog entry on the cost of typesafe code]for details.

You will need to do a few minutes reading on how to format floats and other output because all that lovely %6.3f type specification is gone. The new approach can be a bit more verbose, but after you use it a while you might find you actually like it. To my mind reading: 
<code style="font-size:medium;">
 cout << "Some Value:" << someVariable <<" Some OTher Value:" <<someOtherVariable << endl;
</code>
is actually preferable to the comparable printf statement. The more variables there are the more more I like cout. No lining up specifiers with arguments. Moving arguments around is about as intuitive as you can get. Splitting it up to multiple lines is also as easy as can be. The real downside for most people is the format specification syntax. Here is an example of sending out a float with two places showing after the decimal point.
<code style="font-size:medium;">
 cout << "Test time: "  << setprecision(2) << fixed << setw(6)<< secondsElapsed << " seconds." << endl;
</code>
Not exactly lovely, but hey you had to go look up %6.2f or is it %f6.2 or 6.2%f the first time, you've just gotten used to that ugly big of syntax. 

Just to add a little more resistance to adopting the new approach, someone decided the ios namespace would be in a separate library, so don't forget to 

<code style="font-size:medium;">
 #include <iomanip>
</code>

Here's a series of related examples with the associated output. '''Note''', it's generally not considered good a practice to pull the entire std namespace into scope, with the ''using namespace std;'' statement. The example would have been more cluttered, but more canonical, had it brought just the desired functions into scope with the the following using statement: 
 using std::cout;
 using std::endl;
 ...

[[Image:iomanip_examples.png]]

 
An internet search on "iomanip format" should find you loads of pages that specify all the options.
 
One special mention should be made of trying to output formatted bytes. If you want to output the hex value of a byte you have to make sure the system doesn't treat the byte as the true unsigned char that it really is. In the above example you can see a simple technique is to add +0 to the byte value. When you get more advanced this technique can get in your way. Using ostream_iterator<short> is a better approach, you can learn about that and more from [http://syncor.blogspot.com/2011/03/exploiting-stl-sending-vector-to-cout.html this blog entry].

=== sprintf is to misery as ostringstream is to happiness ===
Or as Scott Meyer's would probably say: Prefer ostringstram to sprintf. Using sprintf requires that you allocate a buffer that is sized appropriately to hold the contents of whatever your sprintf'ing. Using the ostringstream library does not. Overrunning a sprintf buffer can be produce a nasty hard to find error. Why risk it? Using ostringstream will leverage the same skills for formatting that you learned for cout.  Using ostringstream requires that you include <sstream>. Here's an example of how you would use it. Note that you always want to decleare your ostringstream variable on a line by itself. I don't know why but trying to combine the declaration with an output does not work.

<code style="font-size:medium;">
 ostringstream new_property;
 new_property << propName << NAME_VALUE_DELIM << ARRAY_DELIM_START;
</code>

That's it. You don't need to know the type of NAME_VALUE_DELIM or ARRAY_DELIM_START (they're const strings). You don't need to initialize and size a buffer and most importantly you <span style="color:purple">'''don't need to worry about overflowing the buffer'''</span>.  So here's a much more complex example that uses a vector from the STL, don't worry about the iterator syntax just notice how you can easily build up an output string knowing nothing about the return type of a function. That means the return type of the function can change the length or even the type of what is returned and your code won't overflow or trap. It might give ugly output but that's a lot easier to find and fix than a trap that happens ten minutes after a buffer is overflowed. If you want to format the output, see the section above on IO manipulators for cout, the same manipulators will work here.
<code style="font-size:medium;">
 ostringstream new_property;
 new_property << propName << NAME_VALUE_DELIM << ARRAY_DELIM_START;
 vector<string>::iterator iter;
 for (iter = propValues.begin(); iter != propValues.end();)
  {
    <span style="color:green">//NOTE: How would you do this with sprintf - how coupled would this method be to BuildPropertyValue?</span>
      new_property << BuildPropertyValue(*iter, valueType); 
      if (++iter != propValues.end()) new_property << ",";
   }
 new_property << ARRAY_DELIM_END;
</code>
For the insatiably curious, if propName = ArrayPropName and the vector propValues contained two values 42, 420 the result of this bit of code is a well formatted JSON Array that would look something like
ArrayPropName:[42, 420]

The last thing you need to know is how to use the results. If you want a C++ string you can use
<code style="font-size:medium;">
 #include <string>  <span style="color:green">//if you work with C++ strings you want to remember this</span>
 using std::string; <span style="color:green">//and this or using namespace std;</span>
 ...
 new_property.str() <span style="color:green">//new_property is from the previous example</span>
</code>

If you want to work with a c string (prefer working with c++ strings but sometimes you have to use a c string), you just use the C++ string operator c_str(). You can do it all in one statement
<code style="font-size:medium;">
 new_property.str().c_str()
</code>

===convert from strings to numeric types using istringstream===
You've come this far so you may as well quit using the old deprecated atoi() and atof() functions as well. Their major drawback is they return 0 when you pass them a valid string that should convert to 0, but they also return 0 when they fail. istringstream on the other hand lets you know separately if it fails to convert the passed string. While you can use it directly in code, it's much more common to create a simple template to do the work for you. Here's one example: 
<code style="font-size:medium;">
   template<typename T>
   static std::pair<bool,T> ValueFromString(std::string const& theString)
   {
      T value;
      std::istringstream iss (theString);
      if ((iss >> value ).fail() ) return std::make_pair(false, 0);
      return std::make_pair(true, value);
   }
</code>
The above requires that you includ <string>, <utility> (for std::pair) and <sstream>. 
Here's an example of using the template
<code style="font-size:medium;">
   std::pair<bool,int> actual = ValueFromString<int>("123otherjunkIfIwanted");
    <span style="color:green;">//now actual.first will == true and actual.second == 123;</span>
   std::pair<bool,int> wont_work = ValueFromString<int>("somejunk123");  <span style="color:green;">// wont_work.first == false.</span>
</code>
You might prefer a template with an additional parameter that takes a reference to the numeric type and just return a simple boolean instead of a pair. You can also convert hex values by passing. You can see this type of sample at the [http://forums.codeguru.com/showthread.php?231054-C-String-How-to-convert-a-string-into-a-numeric-type CodingGuru site].

==Using Class Member Functions in calls to the OS==
The [[Support_FAQ#What_version_of_the_uCOS_is_the_NetBurner_running|MicroC/OS (aka uCOS) used by NetBurner]] has several calls that take a pointer to a function. For example, you create new tasks with OSTaskCreate() (or OSTaskCreateExt() ). The first parameter to this call is declared as 

:void (*task)(void *pd)

That is, a function pointer that takes a pointer to a void and returns void. The function you pass here is used to kick off the processing for that task. Often it's an event loop of some sort. Since the parameter takes a function pointer you can’t pass a pointer to member function. You can however use the second parameter, which is a pointer to void to pass in a pointer to the instance of the class that has the member function you want to invoke. You need to create a '''static''' wrapper method for the call you  pass to OSTaskCreate. Here's an example:


[[Category: Software]]

<code style="font-size:medium;">
   <span style="color:green;">//Call this Class member function instead of directly calling OSTaskCreate</span>
   void SomeTask::StartupTask()
   {
      const BYTE os_err = OSTaskCreate(InitializeWrapper, this, (void*) &TaskStack[STACK_SIZE], (void*) TaskStack, _taskPriority);
      //check os_err for OS_NO_ERR or OS_PRIO_EXIST or whatever level of error handling you want here
   }
   <span style="color:green;">
   //Wrap what would have been your old static or C function in this method
   //<span style="font-weight:bold;">IMPORTANT NOTE: This method should be declared static in your .h file.</span></span>
   void SomeTask::InitializeWrapper(void* const pTaskObject)
   { 
       SomeTask* p_task_object = static_cast<SomeTask*> pTaskObject;
       p_task_object->Initialize(0);
       //or you can do it one line without the extra variable
       //static_cast<SomeTask*> (pTaskObject)->Initialize(0);
   }
   
   <span style="color:green;">//This is the method that was originally passed directly when
   //it was a C function or a static method.</span>
   void SomeTask::Initialize(const void * const pd)
   {
   //In here you do all the things you want this task to do.
   }
</code>

You would now use this class to start a new task as follows:
<code style="font-size:medium;">
    void UserMain(void * pd)
    {
        <span style="color:green;">//Get an instance of your class however you like. I typically want all my tasks
        //as singletons.</span>
        SomeTask my_task = ServiceProvider::GetSomeTaskAsSingleton();
         my_task.StartupTask();
    }

</code>

I find this particular idiom so useful that I have a TaskBase class that I inherit from for any task my system needs. The TaskBase already has the InitialzeWrapper() static method, and a pure virtual protected function ''virtual void Initialize(const void * const pd)=0;'' In fact, I find it so useful I decided to post the code for this class so you don't have to reinvent the wheel (feel free to improve the wheel though). You should start by reading [http://syncor.blogspot.com/2011/06/easy-way-to-start-task.html ''The Easy Way To Create Tasks''] on Tod Gentille's blog and note there is a gisthub link to the free source code for implementing the base class (including unit tests).

==Leveraging the Power of the C++ Standard Template Library==
One of the first things you'll notice when you start using the STL is that content assist doesn't seem to work on the library. Getting it to work requires setting up the correct include paths. In Eclipse this starts with the Project->Properties menu. Select Path and Symbols on the left, and select the C++ Source Files. Click the Add button followed by the File System button and then navigate to the C:\NBurn\gcc-m68k\m68k-elf\include\c++\4.2.1  folder as shown in the picture. If you use <string> then you'll also want to include the m68k-elf path that is under the 4.2.1 directory. As newer versions of the NNDK are released this path name may change to reflect the version currently in use. 

===Always having the C++ paths included===
As an alternative to the above method, if you are always using C++ features, you can modify the platform properties file so that these files are always included in every project for a specific platform.  For example, on my system I modify the ''C:\Nburn\MOD5272\PROPERTIES'' file. In that file is a line that starts with  '''NBINCLUDE =''' I just add '''"$(NBROOTMINGW)\gcc-m68k\m68k-elf\include\c++\4.2.1\m68k-elf"''' and '''"$(NBROOTMINGW)\gcc-m68k\m68k-elf\include\c++\4.2.1"''' to that line with the proper space separators


[[Image:AddingIncludePaths.png]]

==Using the C Callbacks==
You can put everything in files that use the .cpp extension. When that file contains a C Callback like a function called from a web page via the FUNCTIONCALL tag, ie. &lt;!--FUNCTIONCALL SomeCFunctionCallback --&gt; you just need to remember to use the extern keyword as follows:
<pre style="font-size:medium">
 extern "C"
 {
    void SomeCFunctionCallback (int sock, const char* url)
    {
       //Code that does something goes here.
    }
 }
</pre>
If you only have one function you can skip the brackets, but I get in the habit of always including them, that way if I ever want to add a new function I don't have to go back and add them or repeat the extern "C" qualifier. The extern "C" qualifier by the way is a little misleading. It isn't only for C code, and it isn't even saying that what follows is C code.  It's just a way to tell the C++ compiler, "Hey, don't mangle the name of this function/method, I want to call it with the name I give it.".  This could apply to assembly code or C++ methods that you want to call from C. Of course you can't use it for C++ methods if those methods are overloaded (that's the reason for name mangling) but if they aren't overloaded you can keep them from being mangled so they can be called from your C code.

==Controlling Dynamic Memory Allocation==
One of the simplest ways to control dynamic memory allocation is to completely avoid the new operator. It's a little extreme but it can be done. If you wrote in C and never used malloc(), you can just as easily write in C++ and never use new. You'll do a lot of static initialization and you can write entire classes where all the data and member methods are static. Do see the section below on Initializing Statics. If you don't have any data members you can collect related methods under a namespace instead of inside any class or struct.  ''And while we're on the topic, when you move to C++ you should go cold turkey on malloc() and free(), use new/delete  or new[]/delete[] instead. Also if you don't know the difference between new and new[] and delete and delete[],  you owe it yourself to get a copy of [http://www.amazon.com/exec/obidos/ASIN/0201924889/theswatteamatsyn Effective C++ by Scott Meyers], and read items 3 and 5.'' 

I do find completely avoiding the new operator to be a little extreme. I use the new and new[] operators but usually in a very deterministic fashion. Once my applications have finished initializing it's rare for there to be a new operator invoked. I seldom new up a class inside another class either, I tend to favor manual dependency injection and pass in any classes. This also makes unit testing much easier.

==Caution on Const Correctness==
Generally you want to write your code to be as const correct as possible. One thing to be aware of are variables that are used for special purposes, such as sending to web pages or in the example I'm about to give the char* AppName  variable that most developers set up in main. This special variable name is used by some tools like AutoUpdate to identify the NetBurner found by name. You can define this variable as a pointer to a const char like so:
 const char* AppName="Your Name Here";
'''However''', defining it as a const pointer to a const char may not be a good idea. That is:
 const char* const AppName="Your Name Here";
While this is technically const correct and "better", I suspect it allows the compiler to optimize away the variable. This breaks AutoUpdate and it no longer can see the variable. As mentioned this can also affect variables you are sending to web pages, that have to remain variables and not get optimized away. Otherwise for variables used only in your own code full const correctness is the way to go.

==Efficiency and C++==
Using C++ effectively can really help reduce the maintenance costs on your application. C++ can also drive up the CPU and memory costs on your project. A lot of the NetBurner boards have plenty of horsepower and plenty of RAM and Flash so the costs on those boards are relatively insignificant. If you're running at lower clock speeds or with minimal flash or RAM available then you do need to be aware that some features of C++ can cost you both.  Base class virtual functions have a cost. First, using virtual functions will add one virtual table for the class and a pointer for every instance of the class. The added pointer changes the size of your class, so if you were planning on passing your class to a C function via a pointer things won't work the way they did when you didn't have any virtual functions. In terms of performance there will be one more level of indirection when making a call but that should be relatively insignificant. In fact, if you are using virtual functions, it probably is eliminating either nested if statements or switch statements from your code. You're now using the compiler to write code for you, and it's quite possible your final code will actually be more efficient that the equivalent C code (it will still take more flash and RAM though). Now, if your class only has a few bytes of data and you have lots of instances of the class, adding that one pointer is going to up your memory requirements by a pretty hefty percentage so you do need to be aware of the cost.
===Exceptions and Exception Handling===
Exceptions and exception handling are one of the most obvious additional costs you can incur when moving from C to C++. Of course you don't have to use them.  In fact, the default settings for the NetBurner turn off support for exceptions by default. That's because the stack unwinding and tracking needed just to allow exceptions can add a cost. I suspect most NB developers aren't using them and that was the reason for leaving support off by default. The more try/catch blocks you use the more it costs. It may never be an issue but it's something to keep in mind. Also using exceptions and exception specifications correctly can be trickier than you might think. There are many books available on writing exception safe code. In a lot of software systems exception handling is bubbled up to the top and often as a last resort a message is shown to the user telling them about an uncaught exception or a caught exception that the program can't handle. This can work on a PC-based GUI application. It's not such a good idea on an embedded project. The allure of using exceptions so that you don't have to return error codes and possibly have them ignored is strong. They can make your code easier to write, maintain and debug. They aren't free and they make it possible to add traps that occur that can be hard to track down. (One prime example, is when an exception specification is not adhered to and an unexpected exception is returned. The default behavior when an unexpected exception is detected is to call abort. If you want to use exceptions in your code you owe it to yourself to at least read Items 11 through 15 in Scott Meyer's [http://www.amazon.com/exec/obidos/ASIN/020163371X/theswatteamatsyn More Effective C++]. 
===Templates===
Templates on the other hand seem like a win-win. Only templates that are used get instantiated and take up any room. If you write manual methods for every possible use, you'll probably write some that never get used in some uses of your program and your code will be harder to maintain. There is nothing inherent about templates that make them any more inefficient than hand-written, type-safe code. If you're used to doing things in a type-unsafe way (maybe by passing around a lot of void*) then templates do take more work and probably more code but given sufficient memory, I would say writing safe code is worth the cost.  That's not to say you can't write really suspect templates like this:
<pre style="font-size:medium">
//Don't do things like this!
template <int T1, int T2> struct SomeStruct( SomeStruct(const char param1[T1], const char param2[T2]);}
</pre>
And they probably will waste a lot of room that would be harder to waste without using a template but I'm willing to assume you'll make reasonable uses of templates to improve your code and make it more maintainable and not write them to prove to your boss that you should still be programming in C.

==Initializing Statics - A Cautionary Tale==
There are a couple of ways to get in trouble with non-local static objects (NLSO). They all have to do with when static objects are initialized. On the NetBurner you are used to using UserMain and all your code tends to flow starting from there. It's easy to start thinking of UserMain() as being the same as C's main(). They aren't the same. Even C++'s main() isn't the same as C's main().  The distinction is important when you write any type of non-local static object. I've tested it just to make sure and it's true. Your NLSOs get set up and initialized before UserMain() ever gets called. I suspect strongly that they get initialized before main() gets called but I have no way to prove that.  In fact, from testing results I've gotten, they appear to be called and initialized before the RTOS (see Update Below). This means you don't want to put any RTOS calls in your constructors. If you're new to this game you might wonder why anyone might do that, but functions like OSTimeDly() can get used a lot and it can seem pretty benign. Also some RTOS calls need to made just once to set things up, OSMboxInit() and OSSemInit() come to mind. It's tempting to put calls to these functions in a constructor. If you don't create a static instance of your class you'll never see a problem with this. ''' Don't do it.''' The problem is, the programmer who comes after you might not know that you did this and they might create a static instance of your class. Nothing is more annoying than having a NetBurner trap before the first line of UserMain() gets executed. 

The second trouble area with static objects arises when one NLSO depends on another NLSO. The order of initialization of NLSOs is non-deterministic so this can really lead to hard to diagnose problems. This subject and solution aren't particular to the NetBurner the way calls to the RTOS are. Scott Meyer's discusses this problem in detail in Item 47 of [http://www.amazon.com/exec/obidos/ASIN/0201924889/theswatteamatsyn Effective C++]. Adding to the difficulty in debugging, is the fact that the debugger can't break on any of the NLSO constructors. Remember, your call to set up the debugger is in UserMain, so anything that runs before that has to be debugged with serial port output.

'''Update for NNDK 2.6.022:''' NetBurner has started to modifications in the more recent NNDKs so that ''"the RTOS gets initialized before C++ global contructors"''. This means you can now use RTOS calls in your constructors and if they get turned into statics or globals they'll keep working. Other warnings about NLSOs and the lack of deterministic behavior still applies, so don't throw caution to the wind.

 
Summary: 
Please note that all contributions to user's Wiki! may be edited, altered, or removed by other contributors. If you do not want your writing to be edited mercilessly, then do not submit it here.
You are also promising us that you wrote this yourself, or copied it from a public domain or similar free resource (see user's Wiki!:Copyrights for details). Do not submit copyrighted work without permission!
Save page Show preview  Show changes Cancel | Editing help (opens in new window)
Navigation menu
Create accountLog inPageDiscussionReadEditView history

Search
	 Go	 Search
Navigation
Main Page
Contents
Random page
interaction
About NBwiki
Recent changes
Help
Official NetBurner
NetBurner.com
NetBurner Store
NetBurner Support
NetBurner Forums
Toolbox
What links here
Related changes
Special pages
Page information
Privacy policyAbout user's Wiki!DisclaimersPowered by MediaWiki
