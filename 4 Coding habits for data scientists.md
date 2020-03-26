# Coding habits for data scientists

By: David Tan

Last Updated: 10/21/19

Source: https://www.thoughtworks.com/insights/blog/coding-habits-data-scientists  


### Intro

If you have tried your hand at machine learning or data science, you know that code can get messy, quickly. (He links a jupyter notebook with lots of dataframes printed)  

Typically, code to train ML models is written in jupyter notebooks and it's full of:
- side effects (print statements, pretty-printed data frames, data visualizations)
- glue code without any abstraction, modularisation and automated tests

While this may be fine for notebooks targeted at teaching people about the machine learning process, in real projects it is a recipe for unmaintainable mess. The lack of good coding habits makes code hard ot understand and consequnetly modifying code becomes painful and errorprone. This makes it increasingly difficult for data scientists and developers to evolve their ML solutions.  

In this article, we will share techniques for identifying bad habbits that add to complexity in code as well as habits that can help us partition complexity.  

What contributes to complexity?

> One of the most important techniques for managing software complexity is to design systems so that developers only need to face a small fraction of the overall complexity at any given time. - John Ousterhout  

To tackle complexity, we must first know what it looks like. SOmething is complex when it is composed of interconnected parts. Every time we write code in a way that ads another moving part, we increase complexity and add one more thing to hold in our head.  

While we cannot - and should not try to - escape from the essential complexity of a problem, we often add unnecessary accidental complexity and uncessary cognitive load through bad practices such as:  
- **Not having abstractions** - When we write all code in a single Python notebook or script without abstracting it into functions or classes, we force the reader to read many lines of code and figure out the 'how', to find out what the code is doing.
- **Long functions that do multiple things** - This forces us to have to hold all the intermediate data transformations in our head, while working on one part of the function
- **Not having unit tests** - When we refactor, the only way to ensure that we have not broken anything is to restart the kernel and run the entire notebook. we are forced to take on the complexity of the whole codebase even though we just want to work on one small part of it.  

Complexity is unavoidable, but it can be compartmentalized. In our homes we don't actively organize and rationalize where, why and how we place things, mess accumulates and what should have been a simple task(finding a key) becomes unnecessarily time-consuming and frustrating. The same applies to our codebase. New code is constantly being added for data cleaning, feature engineering, bug fixes, handling new data, and so on. Unless we vigilantly maintain our codebase and continously refactor (and we can not refactor without unit tests), mess and complexity are guaranteed.  


## Habits for reducing complexity

In the remainder of this article, we will share some common bad habits that increase complexity and better habits that help to manage complexity:  
- Keep code clean
- Use functions to abstract away complexity
- Smuggle code out of jupyter notebooks as soon as possible
- Apply test driven development
- Make small and frequent commits

### Keep code clean

Unclean code adds to complexity by making code difficult to understand and modify. As a consequence, changing code to respond to business needs becomes increasingly difficult, and sometimes even impossible.  

One such coding habit (or 'code smell') is dead code. Dead code is code which is executed but whose results are never used in any other compution. Dead code is yet another unrelated thing that developers have to hold in our head when coding. 

Clean code practices have been written about extensively in serveral languages including Python. We have adapted these clean code principles, and you can find them in this clean-code-ml repo:  


[clean code repo general](https://github.com/abiodunjames/Awesome-Clean-Code-Resources)  
[clean code python](https://github.com/zedr/clean-code-python)  
[clean code repo for ML](https://github.com/davified/clean-code-ml)  

### [Design](https://github.com/davified/clean-code-ml/blob/master/docs/design.md)

##### Do not expose your internals (Keep implementation details hidden)  
- Functions and classes simplify our code by abstracting away complicated implementation details and replacing them with a simpler representation - its name.
- when implementation details are all laid bare in jupyter notebooks without any abstractions(functions), we are forced to understand the how's in order to find out what's happening.  
    
  
### [Dispensables](https://github.com/davified/clean-code-ml/blob/master/docs/dispensables.md)
   
##### Avoid print statements, comments and dead code
 
Comments can be problematic in a few ways:  
- If some code needs comments, it is a smell for deeper issues (bad variable naming, violation of single responsibility principle, poor abstraction)
- Comments can grow stale and they can lie
- Comments can make code even harder to understand when there is too much of it

With good variable names, proper abstractions of our code into functiosn with single responsibility and unit tests, we can understand our code without comments.  

    <p>
    Bad: 
    // check to see if empoloyee is eligible for full benefits  
    if (employlee.flags and HOURLY_FLAG) and (employee.age > 65):  
    // do something

    Good:  
    if employee.isEligibleForBenefits():  
    // do something  
    </p>

##### Remove dead code

Dead code is code which is executed but whose result is never used in any other computation. Dead code is yet another unrelated thing that developers have to hold in our head when coding. It adds unnecessary cognitive load.  

If there is code which does not change the result of the program whether it runs or not, then it is not required for the code to run. We should remove it to keep the codebase clean. When we make small and frequent commits, we need not fear losing code. If we ever need those lines of code again, we can easily recover it from the git history.  

One common type of dead code are print statements (even glorified statements such as df.head, df.describe, df.plot). While these offer useful feedback when we are writing code, too much of it makes code hard to read. The team has to visually parse the notebook and spend metnal effort to filter out inconsequential print statements in order to find the code that actually has an effect on the output.  

When our codebase has unit tests, the feedback from unit tests replace the manual/ visual feedback we used to use (df.head) to check if things are working. This allows use to remove these print statementts and make our code more readable.  

### [Variables](https://github.com/davified/clean-code-ml/blob/master/docs/variables.md)

##### Variable names should reaveal intent  

We will read more code than we will ever write. It is important for our code to express intent so that our readers don't have to waste mental effort to figure out puzzles.  

One common culprit in data science code is dataframes. Every dataframe is named as df. In software programming, it is an unusual (and bad) practice to embed information about variable types in the variable name (e.g. we would probably never write string = 'Hello friends'. Instead we would write greetings = 'Hello freinds').

    <p>
    Bad:
    df = pd.read_csv('loans.csv')
    _df = df.groupby(['month']).sum()
    __df = filter_loans(_df, month=12)

    // let's try to calculate total loan amount for december
    total_loan_amount = __df ... # wait, should I use df, _df or __df?


    Good: One rule of thumb on how to name dataframes is to thing about what is in each row. For instance, if each row in the dataframe is a loan, then the dataframe is collection of loan entries. Hence, we could call the dataframe loans.  

    loans = pd.read_csv('loans.csv')

    monthly_loans = loans.groupby(['month']).sum()
    monythly_loans_in_december = filter_loans(monthly_loans, month = 12)


    // let's try to calculate total loan amount for december
    total_loan_amount = monthly_loans_in_decemeber.sum()
    </p>

##### Use meaningful and pronounceable variable names

    <p>
    bad:
    ymdstr = datetime.date.today().strftime("%y-%m-%d")

    good:
    current_date: str = datetime.date.today().strftime("%y-%m-%d")
    </p>

##### Use the same vocabulary for the same type of variables

    <p>
    get_user_info()
    get_client_data()
    get_customer_records()
    
    Good: if the entity is the same, you should be consistent in refering to it in your functions:  
    get_user_info()
    get_user_data()
    get_user_record()
    
    </p>

##### Avoid magic numbers

    <p>
    Bad:
    // what the heck is 86400 for?
    time.sleep(86400)
    
    Good:
    # Extract magic number as variable
    SECONDS_IN_A_DAY = 86400
    
    time.sleep(SECONDS_IN_A_DAY)
    </p>
    
    
##### Use variables to kee code 'DRY' (Don't Repeat Yourself)

Dry stands for Don't Repeat Yourself. If you find yourself changing the samething in multiple places, then that thing which you are changing is a candidate for refactoring.  

    <p>
        Bad: Notice how 'amount' is duplicated in multiple places. if the column name in the data should change, then we would need to waste efforts in finding and replacing amount in multiple places.  
    
    loans = loans.fillna({'amount':0})
    loans.groupby(['amount']).mean().sort_values(by='amount')
    
    Good: Now, should the amount column need to be changed, we can change it in one place:
    
    target_column = 'amount'
    
    loans = loans.fillna({target_column: 0})
    loans.groupby([target_column]).mean().sort_values(by=target_columns)
    
    </p>
    
##### Use explanatory variables

    <p> 
    Bad:

    address = 'One Infinite Loop, Cupertino 95014'
    city_zip_code_regex = r'^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?'
    matches = re.match(city_zip_code_regex, address)

    save_city_zip_code(matches[1], matches[2])

    Not Bad:

    It is better, but we are still heavily dependant on regex 

    address = 'One Infinite Loop, Cupertino 95014'
    city_zip_code_regex = r'^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?'
    matches = re.match(city_zip_code_regex, address)

    city, zip_code = matches.groups()
    save_city_zip_code(city, zip_code)

    Good: decrease dependence on regex by naming subpatterns.  

    address = 'One Infinite Loop, Cupertino 95014'
    city_zip_code_regex = r'^[^,\\]+[,\\\s]+(?P<city>.+?)\s*(?P<zip_code>\d{5})?'
    matches = re.match(city_zip_code_regex, address)

    save_city_zip_code(matches['city'], matches['zip_code'])

    </p>
    
##### Avoid mental mapping

Don't force the reader of oyur code to translate what the variable means. Explicit is better than implicit:

    <p>
    Bad:
    seq = ('Austin', 'New York', 'San Francisco')
    
    for item in seq:
        do_stuff()
        do_some_other_stuff()
        // Wait, what is titem for again?
        dispatch(item)
        
     Good:
     
     locations = ('Austin', 'New York', 'San Francisco')
     
     for location in locations:
         do_stuff()
         do_some_other_stuff()
         dispatch(location)
         
     </p>
     
##### Don't add unneeded context

If your class/object name tells you something, don't repeat that in your variable name.

    <p>
    Bad: 
    
    class Car:
        car_make: str
        car_model: str
        car_color: str
        
    // usage
    car = Car()
    car.car_make
    car.car_model
    car.car_color
    
    Good:
    
    class Car:
        make: str
        model: str
        color: str

    // usage
    car = Car()
    car.make
    car.model
    car.color
    
    </p>
    
### [Functions](https://github.com/davified/clean-code-ml/blob/master/docs/functions.md)

##### Use functions to keep code DRY

The develoepr who learns to recognize duplication, and understands how to eliminate it through proper abstractions (i.e. defining the right functions or methods), can produce much cleaner code than one who continuously infects the application with uncessary repition.  

Every line of code that goes into an application must be maintained, and is a potential source of future bugs. Duplication needlessly bloats the codebase, resulting in more opportunities for bugs and adding accidental complexity into the system. The bloat that duplication adds to the system also makes it more difficult for developers woring with the system to fully understand the entire system, or to be certain that chances made in one location do not also need to be made in other places that duplicate the logic they are workign on. DRY requires that 'every piece of knowledge must have a single, unambiguous, authoritative representation within a system.'  

[97 things every programmer should know](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/)  

    <p>
    Bad: 
    
    decision_tree_model = DecisionTreeClassifier()
    decision_tree_model.fit(X_train, Y_train)
    Y_pred = decision_tree_model.predict(X_test)
    decision_tree_accuracy = round(decision_tree_model.score(X_train, Y_train) * 100, 2)
    print(decision_tree_accuracy)

    random_forest_model = RandomForestClassifier(n_estimators=100)
    random_forest_model.fit(X_train, Y_train)
    Y_pred = random_forest_model.predict(X_test)
    random_forest_model.score(X_train, Y_train)
    random_forest_accuracy = round(random_forest_model.score(X_train, Y_train) * 100, 2)
    print(random_forest_accuracy)

    gaussian_model = GaussianNB()
    gaussian_model.fit(X_train, Y_train)
    Y_pred = gaussian_model.predict(X_test)
    gaussian_accuracy = round(gaussian_model.score(X_train, Y_train) * 100, 2)
    print(gaussian_accuracy)

    Good:  
    
    def train_model(ModelClass, X_train, Y_train, **kwargs):
    model = ModelClass(**kwargs)
    model.fit(X_train, Y_train)

    accuracy_score = round(model.score(X_train, Y_train) * 100, 2)
    print(f'accuracy ({ModelClass.__name__}): {accuracy_score}')

    return model, accuracy_score


    decision_tree_model, decision_tree_accuracy = train_model(DecisionTreeClassifier, X_train, Y_train)
    random_forest_model, random_forest_accuracy = train_model(RandomForestClassifier, X_train, Y_train, n_estimators=100)
    gaussian_model     , gaussian_accuracy      = train_model(GaussianNB, X_train, Y_train)
    
    Tip: Notice how symmetry of the 3 code blocks in the back example made it easier for us to identify and refactor the duplicated code? One useful practice in eliminating duplication is to first make the duplication as obvious as possible. This makes it easier for us to identify opportunities for extracting the duplication into their appropriate homes. 
    </p>
    
##### Functions should do one thing

This is by far the most important rule in software engineering. When functions do more than one thing, they are harder to compose, test, and reason about. When you can isolate a function to just one action, they can be refactored easily and your code will read much cleaner. If you take nothing else away from this guide other than this, you will be ahead of many developers.  

    <p>
    Bad:
    
    def email_clients(clients: List[Client]):
    """Filter active clients and send them an email.
    """
    for client in clients:
        if client.active:
            email(client)
            
    Good: 
    
    def get_active_clients(clients: List[Client]) -> List[Client]:
    """Filter active clients.
    """
    return [client for client in clients if client.active]


    def email_clients(clients: List[Client, ...]) -> None:
    """Send an email to a given list of clients.
    """
    for client in clients:
        email(client)
        
    Even better: using generators
    
    def active_clients(clients: List[Client]) -> Generator[Client]:
    """Only active clients.
    """
    return (client for client in clients if client.active)


    def email_client(clients: Iterator[Client]) -> None:
        """Send an email to a given list of clients.
        """
        for client in clients:
            email(client)
            
    </p>
    
###### Functions should only be one level of abstraction

When you have more than one level of abstraction, your function is usually doing too much. Splitting up functions leads to reusability and easier testing.  

    <p>
    
    Bad: 
    
    def parse_better_js_alternative(code: str) -> None:
        regexes = [
        //...
    ]

    statements = regexes.split()
    tokens = []
    for regex in regexes:
        for statement in statements:
            //...

    ast = []
    for token in tokens:
        // Lex.

    for node in ast:
        // Parse.
        
    Good: 
    
    def parse_better_js_alternative(code: str) -> None:
    tokens = tokenize(code)
    syntax_tree = parse(tokens)

    for node in syntax_tree:
        # Parse.


    def tokenize(code: str) -> list:
        REGEXES = [
            # ...
        ]

        statements = code.split()
        tokens = []
        for regex in REGEXES:
            for statement in statements:
               # Append the statement to tokens.

        return tokens


    def parse(tokens: list) -> list:
        syntax_tree = []
        for token in tokens:
            # Append the parsed token to the syntax tree.

        return syntax_tree
    </p>
    
    
##### Function names should say what they do

    <p>
    Bad:
    class Email:
    def handle(self) -> None:
        # Do something...

    message = Email()
    # What is this supposed to do again?
    message.handle()
    
    
    Good: 
    
    class Email:
    def send(self) -> None:
        """Send this message.
        """

    message = Email()
    message.send()
    
    </p>

##### Use type hints to improve readability

using type hints can make your code more readable and reasonable. Your development experience will also be improved because your IDE will be able to give you better auto-complete suggestions about functions/method names and parameters. 

    <p>
    
    Bad:
    
    def categorize_continuous_values(pd_series, num_bins):
    // do something
    
    Good: with type hints, we can name our variables sensibly, and the IDE now offers better autocompletion and makes us more productive and less error-prone.  
    
    def categorize_continuous_values(values: pd.Series, num_bins: int):
    // do something
    
    It is important to note that type hints are meant to be entirely ignored by the python runtime, and are checked only by 3rd party tools like mypy and pycharms integraded checker.
    
    [Python_annotation/type checkers](https://www.bernat.tech/the-state-of-type-hints-in-python/)
    
    </p>
    
##### Avoid side effects

A function produces a side effect if it does anything other than take a value in and return another value or values. For example, a side effect could be writing to a file, modifying some global variable, or accidentally wiring all your money to a stranger.  

Now, you need to hav eside effects in a program on occasion - for example, like in the previous example, you might need to write to a file, In these cases, you should centralize and indicate where you are incorporating side effects. Don't have several functions and classes that write to a particular file - rather, have on (and only on) service that does it.  

The main point is avoid common pitfalls like sharing state between objects without any structure, using mutable data types that can be written to be anything, or using an instance of a class, and no centralizing where your side effect occurs. If you can do this, you will be happier than that vast majority of other programmers.  

    <p>
    
    Bad:

    // Global variable referenced by following function.
    #//If another function used this name, now it'd be an array and could break.
    name = 'Ryan McDermott'

    def split_into_first_and_last_name() -> None:
        global name
        name = name.split()

    split_into_first_and_last_name()
    print(name)  // ['Ryan', 'McDermott']

        // calling this function the second time will throw AttributeError: 'list' object has no attribute 'split'
        split_into_first_and_last_name()

    Good: 


    def split_into_first_and_last_name(name: str) -> None:
        return name.split()

    name = 'Ryan McDermott'
    new_name = split_into_first_and_last_name(name)

    print(name)  // 'Ryan McDermott'
    print(new_name)  // ['Ryan', 'McDermott']
    
    </p>
    
##### Avoid unexpected side effects on values passed as function parameters

We can unexpectedly change the values passed to our functions, even though our functions appear to be pure.  

This will happen when we pass non-primitive objects (e.g. lists, dictionaries, isntances of classes, pandas dataframes) to a function because in Python (and indeed many other languages), non-primitive objects are passed by reference.

    <p>
    Bad: 
    
    import pandas as pd

    original = pd.DataFrame({
        'values': [1,2,3],
    })

    def multiply_column_by_10(df, column_name):
        df['multiplied column'] = df[column_name] * 10

        return df

    new = multiply_column_by_10(original, 'values')

    original.head() # surprise! original dataframe is mutated and now it has
    
    Good:
    
    import pandas as pd

    original = pd.DataFrame({
        'values': [1,2,3],
    })

    def multiply_column_by_10(df, column_name):
        df = df.copy()
        df['multiplied column'] = df[column_name] * 10

        return df

    new = multiply_column_by_10(original, 'values')

    original.head() # original dataframe is not mutated
    
    </p>
    
##### Function arguments (2 or fewer ideally)

Limiting the amount of function parameters is incredibly important because it makes testing your function easier. Having more that three leads to combinatorial explosion where you have to test tons of different cases with each seperate arguments.  

One or two arguments is ok, and three should be avoided. Anything more than that should be consolidated. Usually if oyu have more than two arguments that your function is trying to do too much. In case where it is not, most of the time a higher-level object will suffice as an argument.  

    <p>
    
    Bad: 
    
    def create_menu(title, body, button_text, cancellable):
    
    Good:
    
    Class Menu:
        def __init__(self, config: dict):
            title = config["title"]
            body = config["body"]
            // ...
    menu = Menu(
        {
            "title": "My Menu",
            "body": "Something about my menu",
            "button_text": "OK",
            "cancellable": False
        }
    )
    
    Also good:

    class MenuConfig:
        """A configuration for the Menu.

        Attributes:
            title: The title of the Menu.
            body: The body of the Menu.
            button_text: The text for the button label.
            cancellable: Can it be cancelled?
        """
        title: str
        body: str
        button_text: str
        cancellable: bool = False


    def create_menu(config: MenuConfig):
        title = config.title
        body = config.body
        # ...


    config = MenuConfig
    config.title = "My delicious menu"
    config.body = "A description of the various items on the menu"
    config.button_text = "Order now!"
    # The instance attribute overrides the default class attribute.
    config.cancellable = True

    create_menu(config)
    Fancy:

    from typing import NamedTuple


    class MenuConfig(NamedTuple):
        """A configuration for the Menu.

        Attributes:
            title: The title of the Menu.
            body: The body of the Menu.
            button_text: The text for the button label.
            cancellable: Can it be cancelled?
        """
        title: str
        body: str
        button_text: str
        cancellable: bool = False


    def create_menu(config: MenuConfig):
        title, body, button_text, cancellable = config
        # ...


    create_menu(
        MenuConfig(
            title="My delicious menu",
            body="A description of the various items on the menu",
            button_text="Order now!"
        )
    )
    Even fancier:

    from dataclasses import astuple, dataclass


    @dataclass
    class MenuConfig:
        """A configuration for the Menu.

        Attributes:
            title: The title of the Menu.
            body: The body of the Menu.
            button_text: The text for the button label.
            cancellable: Can it be cancelled?
        """
        title: str
        body: str
        button_text: str
        cancellable: bool = False

    def create_menu(config: MenuConfig):
        title, body, button_text, cancellable = astuple(config)
        # ...


    create_menu(
        MenuConfig(
            title="My delicious menu",
            body="A description of the various items on the menu",
            button_text="Order now!"
        )
    )
    
    </p>
    
##### Use default arguments instead of short circuiting or conditionals

Tricky:

Why write:
    
    <p>
    
    def create_micro_brewery(name):
    name = "Hipster Brew Co." if name is None else name
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
... when you can specify a default argument instead? This also makes ist clear that you are expecting a string as the argument.

Good:

def create_micro_brewery(name: str = "Hipster Brew Co."):
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
    
    </p>
    
##### Don't use flags as function parameters

Flags tell your user that the function does more than one thing. Functions should do one thing. Split your functions if they are following different code paths based on a boolean.

    <p>
    
    Bad:

    from pathlib import Path

    def create_file(name: str, temp: bool) -> None:
        if temp:
            Path('./temp/' + name).touch()
        else:
            Path(name).touch()
    Good:

    from pathlib import Path

    def create_file(name: str) -> None:
        Path(name).touch()

    def create_temp_file(name: str) -> None:
        Path('./temp/' + name).touch()
        
    </p>
    
### Use functions to abstract away complexity

Functions simplify our code by abstracting away complicated implementation details and replacing them with a simpler representation - its name.  

What did we gain by abstracting away the complexity into functions?  
- **Readability**
    - We just have to read the interface to know what it is doing.
- **Testability**
    - Because it is now a function, we can easily write a unit test for it. If we accidentally change its behaviour, the unit tests will fail and give us feedback within miliseconds
- **Reusability**
    - To repeat the same transformation on any column, we just need one line (not seven lines) of code
    
When we refactor to functions, our notebook can be simplified and made more elegant. 

Our mental overhead is drastically reduced with good refactoring. We are no longer forced to process many many lines of implementation details to understand the entire flow. Instead the abstraction (functions) abstract away the complexity and tell us what they do, and save us from havingto spend mental effort figuring out how they do it.  

### Smuggle code out of jupyter notebooks as soon as possible

An interior design, there is a concept (the law of flat surfaces) which states that 'any flat surface within a home or office tends to accumulate clutter.' Jupyter notebooks are the flat surface of the ML world.  

Sure, jupyter notebooks are great for quick prototyping. But it is where we tend to put many things - glue code, print statements, glorified print statements (df.describe()) or df.plot(), unused import statements and even stack traces. Despite our best intentions, so long as the notebooks are there, mess tends to accumulate.  

Notebooks are useful because they give us fast feedback, and that is often what we want when we are given a new dataset and a new problem. However, the longer the notebooks become, the harder it is to get feedback on whether our changes are working.  

In contrast, if we had extracted our code into functions and Python modules and if we have unit tests, the test runner will give us feedback on our changes in a matter of seconds, even when there are hundreds of functions.  

Hence, our goal is to move code out of notebooks into Python modules and packages as early as possible. That way they can rest within the safe confines of unit tests and domain boundaries. THis will help to manage complexity by providing a structure for organizing code and tests logically and make it easier for us to evolve our ML solution.  

So how do we move code out of jupyter notebooks?  

Assuming you alread have your code in a Jupyter notebook, you can follow this process: 
![refactoring cycle](https://insights-images.thoughtworks.com/Screenshot202019101620at20100518_df186104d85da2fc7cd6080b96327600.png)

### How to refactor a jupyter notebook

##### What is refactoring

Refactoring is changing code to make it easier to understand and modify without changing its observable behavior  

Refactoring without tests is hard. And typically, when we write Jupyter notebooks, we usually just bang out code without writing tests.  

Nonetheless, since this is the predicament which we find ourseleves in, we need to find a way towards code that is unit-tested and properly abstracted, so that we can have maintainable, readable and extensible codebase. So that we can reamin productive even as we are confronted with new business requirements and new data.  

##### The refactoring process

Before we start refactoring:  
- ensure the notebooks will run from start to end
    - even better if you can add an 'integration' or 'functional' test of the code
- Make a copy of the original notebook (for comparing the end result later)
    - This frees us up from any emotional attachment and allows us to ruthlessly clean up the code
- Remove print statements (print, df.head, df.plot)
    -this removes noise and visual clutter, and makes the next step exponentially easier

The refactoring cycle:
1. Identify a block of code that can be extracted into a pure function
2. Pseudo-TDD (test-driven development)
    - Run the unit test in watch mode
    - Write a unit test for the code block
    - Create a Python module and define a function. Move existing implementation from notebook into that function
    - make the test pass
3. In the notebook, replace original code blocks with the newly defined function
4. Restart and run entire notebook (unforutnately, unt we have sufficient unit tests, we will still need manual 'integration; tests for the time being')
5. if possible, refactor functions some more
6. commit your changes
    - git add
    - git commit -m 'your commit message'
7. Rinse and repeat

##### Some tips:

- When refactoring, don't change the program's observable behavior (remember the two hats)
    - This means you either Refactor and commit
    - Or you add a function and commit
- Keep tests in watch mode. Test continuously during each refactoring
    - Testing after each change means that when i make a mistake, I only have a small change to consider in order to spot the error, which makes it far easier to find and fix
- Make small and frequent commits
- don't try big bang refactoring. Refactoring is not an activity that is seperated from programming. We refactor as we go.
- When to refactor - Three strikes rule - Martin Fowler
    - The first time you do something, you just do it
    - The second time you do something similar, you wince at the duplication, but you do it
    - The third time you do something similar, you refactor




The details of clean step in this process (e.g. how to run a tests in watch mode) can be found in the clean-code-ml repo.  

### Apply test-driven development

So far, we have talked about writing tests after the code is already written in the notebook. This recommendation is not ideal, but it is still far better than not having unit tests.  

There is a myth that we cannot apply test-driven development(TDD) to machine learning projects. To us, this is simply untrue. In any machine learning project, most of the code is concerned with data transformations (data cleaning, feature engineering) and a small part of the code base is actually machine learning. Such data transformations can be written as pure functions that return the same output for the same input, and as such, we can apply TDD and reap its benefits. For example, TDD can help us break down big and complex data transformations into smaller bite-size problems that we can fit in our head, one at a time.  

As for testing that the actualy machine learning part of the code works as we expect it to, we can write functional tests to assert that the metrics of the model (e.g. accuracy, precision, etc) are above our expect threshold. In other words, thse test assert that the model functions according to our expections (hence the name, functional test). Here is an example of such test:  

<p>
    
    import unittest 
from sklearn.metrics import precision_score, recall_score
from src.train import prepare_data_and_train_model

class TestModelMetrics(unittest.TestCase):
  def test_model_precision_score_should_be_above_threshold(self):
    model, X_test, Y_test = prepare_data_and_train_model()
    Y_pred = model.predict(X_test)

    precision = precision_score(Y_test, Y_pred)

    self.assertGreaterEqual(precision, 0.6)

  def test_model_recall_score_should_be_above_threshold(self):
    model, X_test, Y_test = prepare_data_and_train_model()
    Y_pred = model.predict(X_test)

    recall = recall_score(Y_test, Y_pred)

    self.assertGreaterEqual(recall, 0.6)
</p>

### Make small and frequent commits

When we don't make small and frequent commits, we increase metnal overhead. While we are working on this problem, the changes for earlier ones are still shown as uncommitted. This distracts us visually and subconsciously; it makes it harder for us to focus on the current problem.  

When we make small and frequent commits, we get the following benefits:  
- reduced visual distractions and cognitive laod
- we do not need to worry about accidentally breaking working code if it is already been commited
- in addition to red-green-refactor, we can also red-red-red-rever. if we were to inadvertently break something, we can easily fall back checkout to the latest commit, and try again. This saves us from wasting time undoing problems that we accidentally created when we were trying to solve the essential problem.   

So, how small of a commit is small enough? Try to commit when there is a single group of logically related changes and passing tests. One technique is to look out for the word 'and' in our commit message. 'Add exploratory analysis and split sentences into tokens and refactor model training code'. Each of these three changes could be split up into three logical commits. In this situation, you can use git add --patch to stage smaller batche to be commited.  

### Conclusion

> 'Im not a great programmer; I'm just a good programmer with great habits.' - Kent Back, pioneer of Extreme Programming and xUnit testing frameworks  

These are habits that have helped us manage complexity in machine learning and data science projects. We hope it helps you becoming more agile and productive in your data projects as well.  

If you are interest in examples of how organizations can adopt continuous delivery practices to help machine learning practioners and projects be agile see:

[agile material](https://www.thoughtworks.com/insights/articles/intelligent-enterprise-series-cd4ml)  
[agile material 2](https://www.thoughtworks.com/insights/blog/getting-smart-applying-continuous-delivery-data-science-drive-car-sales)

