# Summary

- [Summary](#summary)
- [Notes](#notes)
- [Structure](#structure)
  - [Actions](#actions)
    - [Arguments](#arguments)
    - [Tags](#tags)
  - [Factories](#factories)
    - [Faker Library](#faker-library)
  - [Template](#template)
- [Base.robot](#baserobot)
- [Database.robot](#databaserobot)
- [SignupRequired.robot](#signuprequiredrobot)
- [Temp.robot](#temprobot)
  - [Loops](#loops)
  - [Keywords](#keywords)
- [Login.robot](#loginrobot)
- [BeGeek.robot](#begeekrobot)
  - [Filling type input elements](#filling-type-input-elements)
  - [Filling type select elements](#filling-type-select-elements)
  - [Submitting geek form and message validation](#submitting-geek-form-and-message-validation)
  - [Improvements](#improvements)
    - [Test template improvement](#test-template-improvement)
- [Smoke test](#smoke-test)
- [Screen Resolution](#screen-resolution)
- [Clearing HTML forms](#clearing-html-forms)
- [Dealing with conditional elements (IFs)](#dealing-with-conditional-elements-ifs)
- [Challenge 1 - module PRO](#challenge-1---module-pro)
  - [Automate 3 new test cases inside the login suite:](#automate-3-new-test-cases-inside-the-login-suite)
- [API Testing](#api-testing)
  - [Thunder Client](#thunder-client)
    - [Collections](#collections)
    - [HTML Methods](#html-methods)
    - [Status Codes](#status-codes)
    - [Tests](#tests)
      - [Validating token](#validating-token)
      - [Front-end x back-end](#front-end-x-back-end)
  - [Automating API Tests with robot](#automating-api-tests-with-robot)
    - [Alternative scenarios (sad paths)](#alternative-scenarios-sad-paths)
  - [Sessions.robot](#sessionsrobot)
  - [Users.robot](#usersrobot)
    - [Validating ID](#validating-id)
- [Usefull terminal commands](#usefull-terminal-commands)
  - [Git](#git)
  - [Linux](#linux)
  - [chmod +x run.sh](#chmod-x-runsh)
  - [Request Library](#request-library)
  - [json=${variable} - convert the request content-type to json](#jsonvariable---convert-the-request-content-type-to-json)
- [Important links](#important-links)
  - [Pabot](#pabot)
    - [Pabot screenshot path improvement](#pabot-screenshot-path-improvement)
  - [Typora](#typora)

---

# Notes

Here we're going to automate the web application [Getgeeks](https://geeks-web-andre.fly.dev/) geratered using Fly that is connect to a PostgreSQL database. We're also using ElephantSQL to manage our database.

We'll use Gherkin to define our test scenarios following Behaviour Driven Development (BDD) standards.

BDD is a colaborative specification technique where development driven scenarios are written considering the final users' point of view in order to give a better undertanding and clear informations to facilitate development and also testing after all. BDD is used to guide the development, not only testing automation and should be defined before starting the development.

**Example:** [Register notes](docs/signup.md)

```md
> Sendo um visitante que deseja contratar servi??os de TI
> Posso fazer o meu cadastro
> Para que eu possa buscar por prestadores de servi??os (Geeks)

##### Cen??rio: Cadastro de cliente

Dado que acesso a p??gina de cadastro
Quando fa??o o meu cadastro com o nome completo, e-mail e senha
Ent??o vejo a mensagem de boas vindas:
"Agora voc?? faz parte da Getgeeks. Tenha uma ??tima experi??ncia."
```

We can list here our Sad scenarios, those cases that can lead to erros, as well:

```md
##### Cen??rio: E-mail duplicado

Dado que acesso a p??gina de cadastro
Porem o meu e-mail j?? foi cadastrado
Quando fa??o o meu cadastro com o nome completo, e-mail e senha
Ent??o vejo a mensagem de alerta:
"Oops! J?? temos um usu??rio com o e-mail informado."

##### Cen??rio: Email com formato incorreto

Dado que acesso a p??gina de cadastro
Quando fa??o o meu cadastro com um email incorreto
Ent??o vejo a mensagem de alerta "O email est?? estranho"

##### Cen??rio: Campos obrigat??rios

Dado que acesso a p??gina de cadastro
Quando submeto o cadastro sem preencher o formul??rio
Ent??o devo ver uma mensagem informando que todos os campos s??o obrigat??rios
```

---

# Structure

We'll use _PascalCase_ in here to define files and _snake_case_ to define methods,

## Actions

It's very important to consider the creation of actions for repetitive steps from the start in order to arrange the code properly and make it easier to use the same actions in different scenarios.

**Note:**

<p>One way of doing it is by considering each step of the BDD as an action.</p>

**For example:** [Actions](resources/Actions.robot)

- Dado que acesso a p??gina de cadastro

```
*** Keywords ***
Go to signup form
    Go To    https://geeks-web-andre.fly.dev/signup

    Wait For Elements State    .signup-form h1 >> text="Cadastro"    visible    5

```

- Quando fa??o o meu cadastro com o nome completo, e-mail e senha

```
# When entering input information
    # Checkpoint - going to register page
    Fill Signup Form    ${user}

# And submiting valid information
    Submit Signut Form
```

- Ent??o vejo a mensagem de boas vindas:

```
# Then I should register the user and see the welcome message
    User Should Be Registered
```

**Important**

<p>One important thing to remember is that as actions will be defined inside a different file you need to add this as a Resource inside the Test case file and inside the Base file as well in order to integrate all the information.</p>

### Arguments

Arguments can be used to reduce reusage and facilitate code maintenance once it's necessary to update and/or change only one element. For example:

```
Modal Content Should Be
    [Arguments]    ${expected_text}

    ${title}    Set Variable    css=.swal2-title
    ${container}    Set Variable    css=.swal2-html-container

    Wait For Elements State    ${title}    visible    5
    Get Text    ${title}    equal    Oops...

    Wait For Elements State    ${container}    visible    5
    Get Text    ${container}    equal    ${expected_text}
```

### Tags

Tags can be used to identify one specific test case and allow you to run individual test cases by calling they're tags using:
`-i tag_name` before the test case name. For example:

```
Duplicate user
    [Tags]    dup_email

    Go To Signup Form
    Fill Signup Form    ${user}
    Submit Signut Form
    Modal Content Should Be    J?? temos um usu??rio com o e-mail informado.
```

## Factories

Factories are used to avoid the need of entering all testing set of information individually for each element. It works by defining a factory (factory_user) and then defining all arguments (test dough) inside a variable (user) which will be returned as response. **For example:** [Users factory](resources/factories/Users.py)

```
# Defining test dough

def factory_user():
    user = {
        'name': 'Tony',
        'lastname': 'Stark',
        'email': 'tony@stark.com',
        'password': 'pwd123'
    }

    return user
```

Then we add this variable to an action, defining which argument to use for each case:

```
    Fill Text    input[id="name"]    ${user}[name]
    Fill Text    input[id="lastname"]    ${user}[lastname]
    Fill Text    input[id="email"]    ${user}[email]
    Fill Text    input[id="password"]    ${user}[password]
```

And finally we integrate this factory and insert it inside our test cases:

```
*** Test Cases ***
Register a new user
# Entering test data mass
    ${user}    Factory User

# Once I enter register page
    Go To Signup Form

# When entering input information
    # Checkpoint - going to register page
    Fill Signup Form    ${user}
```

### Faker Library

Faker is a python package used to create dynamic arguments, making it possible to test scenarios with different sets of information automatically created. This can be really usefull for scenarios where you need unique data like registering users, validating emails and so on.

To use it we just need to configure faker inside our factory file:
[Using Faker Library](https://pypi.org/project/Faker/)

```py
# Defining test dough

from faker  import  Faker
fake = Faker()

def factory_user():
    user = {
        'name': fake.first_name(),
        'lastname': fake.last_name(),
        'email': fake.free_email(),
        'password': 'pwd123'
    }

    return user
```

And then run the project again now using dynamic mass of test.

**Obs.:** using faker can create a massive amont of information inside the database, in order to avoid this we install 2 libraries:

- `pip install robotframework-databaselibrary` - Gives access to the database through robot
- `pip3 install psycopg2` - Connect to the postgreSQL database natively

## Template

It is possible to define several steps inside a keyword and then use template to indicate that the test case should follow those steps one at a time following exactly what is defined inside the keyword. This allows us to repeat several scenarios changing only the test variables.

To create a template we first need to define a new keyword with all desired steps:

```
*** Keywords ***
Signup With Short Password
    [Arguments]    ${short_pass}

    ${user}    Create Dictionary
    ...    name=Test    lastname=User
    ...    email=test@email.com
    ...    password=${short_pass}

    Go To Signup Form
    Fill Signup Form    ${user}
    Submit Signut Form
    Alert Span Should Be    Informe uma senha com pelo menos 6 caracteres
```

And then, call this template inside the test case that should reproduce the template behavior, informing all inputs as well:

```
Short Password
    [Tags]    attempt_signup    short_pass
    [Template]    Signup With Short Password
    1
    12
    123
    1234
    12345
    a
    a2
    ab3
    ab3c
    a23bc
    -1
    acb#1
```

**Important:** template will only work if all conditions are exactly equal to one another. Another way to add these repetitive structures, is by using suite setups that will be explained better in the Signup.robot section.

---

# Base.robot

Here's all the base data for testing

It's possible to add dynamic variables to Robot's Command Line Interface (CLI) by defining a variable and calling it with -v VARIABLE:value inside the console:

1. Here we first defined the variables inside de [Base File](resources/Base.robot)

```
*** Keywords ***
Start Session
    New Browser    ${BROWSER}    headless=${HEADLESS}    slowMo=00:00:00.3
    New Page    ${BASE_URL}
    Set Viewport Size    1440    900
```

2. Then added it to our console shortcut inside our [Terminal Shortcut File](run.sh) with -v

```
robot -d ./logs -v BROWSER:chromium -v HEADLESS:true -i smoke tests
```

**Note:** This is important while running continuos testing with Jenkins but will only work while running the application through the terminal. Running this with VS Code for example will return error because ${BROWSER} and ${HEADLESS} variables aren't defined inside the code.

# Database.robot

Here we're creating a task that will return the application to default and then configurate all required information to run our tests. Meaning that after running all database entered information will be deleted and the application will run as if it was the first time.

**Important:** always disconnect from the database after finishing the task. (BEST PRACTICE)

**Note:** always consider password_dash instead of password while dealing with databases as it uses encripted passwords

For the register test cases we were using a forced hardcoded password inside the database.

```
Insert User
    [Arguments]    ${u}
# password must be encripted to a successfull login

    ${q}    Set Variable
    ...    INSERT INTO public.users (name, email, password_hash, is_geek) values ('${u}[name] ${u}[lastname]', '${u}[email]', '${u}[password]', false)

    Execute SQL String    ${q}
```

While creating the login test case it won't work because of the password integrity, so we'll need to now update our test to consider the encripted password instead of the hardcoded one.

---

# SignupRequired.robot

The use of templates are not recommended in same cases because it will run all the steps as they come. Using required fields as example the test case will open a new browser tab and start the hole process all over for each conditions, instead of validation all of them at once, even considering that they're all being shown in the same page.

One way to deal and improve templates is by defining a hole test suite using test cases and defining all steps inside a keyword. The key here is to use the `Suite Setup` as start point instead of the previous Test Setup. It will allow all test cases to run one after another without the need to close and open tabs every time.

**Example:**

```
*** Settings ***
Documentation       Signup Test Suite

Resource            ../resources/Base.robot

Suite Setup         Signup Without Fill Form    # execute the setup once for suite and not for test as used before
Test Teardown       Finish Section

*** Test Cases ***
Name is required
    Alert Span Should Be    Cad?? o seu nome?
Lastname is required
    Alert Span Should Be    -Remover esse texto- E o sobrenome?
Email is required
    Alert Span Should Be    O email ?? importante tamb??m!
Password is required
    Alert Span Should Be    Agora s?? falta a senha!

# Testing all forms all together using form
*** Keywords ***
Signup Without Fill Form

    Start Section
    Go To Signup Form
    Submit Signut Form
```

---

# Temp.robot

Tabulation is essential for lists functioning

By using `@{name}` robot will consider this as a temporary element. Yet to call it as a variable we need to use the `${name}`

## Loops

It is possible to use FOR to create a loop that will be executed until there's no more element available to use

**Example:**

```
Working with Lists

    @{avengers}    Create List    Tony Stak    Kamalakhan    Steve Rogers

    Append To List    ${avengers}    Hulk

      FOR    ${a}    IN    @{avengers}
          Log To Console   ${a}

      END
```

**Note:** Here all individual list elements 'a' inside the avengers list will be logged into console one by one until there's no more elements available.

## Keywords

Append To List allows us to add a new element to the list. Example:

```
Working with Lists

    @{avengers}    Create List    Tony Stak    Kamalakhan    Steve Rogers

    Append To List    ${avengers}    Hulk

    Log To Console  ${avengers}[0]
    Log To Console  ${avengers}[1]
    Log To Console  ${avengers}[2]
```

**Important:** to use this arguments we need to import the library Collections from Robot

---

# Login.robot

Tooltip cannot be inspected as it isn't part of the code, instead it is a browser navigator response that cannot be used as selector. This happens when the input type is defined as email and one solution is to:

Create a new keyword to validate the element type inside the code:

```
Should Be Type Email
    Get Property    id=email    type    equal    email
```

And adding it to our test case:

```
Incorrect Email
    [Tags]    incorrect_email

    ${user}    Create Dictionary    email=invalid.format.com    password=pwd123

    Go To Login Page
    Fill Credentials    ${user}
    Submit Credentials
    Should Be Type Email
```

**Important:** global variables should be used wisely, but one best practice is to create a global variable for global elements that repeat themselves throughout the code. In this example we created the `${INPUT_PASS}` global variable as well just for a visual purpose.

---

# BeGeek.robot

Do Login variable is added in order to facilitate login functionality. The login helper cannot be used inside login test suite because we're validating the login process step by step, then we'll need separated steps to test all functionality coverage.

For the geek form we'll need to fill 5 different fields, in order to do it properly we'll update our test dough generator factory_user_be_geek by adding the sub-dictionary geek_profile inside Users.py:

```
def factory_user_be_geek():
    return {
        'name': 'Kim',
        'lastname': 'Dotcom',
        'email': 'kim@dot.com',
        'password': 'pwd123',
        'geek_profile': {
            'whats': '11999999999',
            'desc': 'Seu computador est?? lento? Reiniciando do nada? Talvez seja um v??rus, ou algum hardware com defeito. Posso fazer a manuten????o no seu PC, formatando, reinstalando o SO, trocando algum componente f??sico e porque n??o remover o baidu ou qualquer outro malware.',
            'printer_repair': 'Sim',
            'work': 'Remote',
            'cost': '100'
        }
```

To define the elements name we normally use those defined inside the HTML, for example:

Description's field HTML:

```html
<div class="sc-kstqJO hxBQUd">
  <label for="desc">Descri????o</label><textarea id="desc"></textarea>
</div>
```

**Note:** here the id selector "desc" is used.

## Filling type input elements

```
Fill Geek Form
    [Arguments]    ${geek_profile}

    Fill Text    id=whatsapp    ${geek_profile}[whats]
    Fill Text    id=desc    ${geek_profile}[desc]

    Fill Text    id=cost    ${geek_profile}[cost]
```

## Filling type select elements

```
Fill Geek Form
    [Arguments]    ${geek_profile}

    Select Options By    id=printer_repair    value    ${geek_profile}[printer_repair]
    Select Options By    id=work    value    ${geek_profile}[work]
```

## Submitting geek form and message validation

First we'll define new keywords for the desired steps:

```
Submit Geek Form
    Click    css=button >> text=Quero ser um Geek

Geek Form Should Be Success
    ${expected_message}    Set Variable
    ...    Seu cadastro est?? na nossa lista de geeks. Agora ?? s?? ficar de olho no seu WhatsApp.

    Wait For Elements State    css=p >> text=${expected_message}    visible    5
```

Finally we'll add it to our, now complete, be a geek test case:

```
*** Test Cases ***
Be a Geek
    # Given that I have a common user
    ${user}    Factory User Be Geek

    # And I log in the plataform Getgeeks
    Do Login    ${user}

    # When submitting the form to become a Geek (service provider)
    Go To Geek Form
    Fill Geek Form    ${user}[geek_profile]
    Submit Geek Form

    # Then I should see the success message
    Geek Form Should Be Success
```

**Note:** the gherkin BDD usage here isn't considering the automation but the expected scenario behavior, that then guide the automation development. Differently from cucumber that needs structured language to define the automation steps, this method gives a more flexible way of steps description without lossing functionality and flow details.

## Improvements

For every alternative scenario where the user don't actually successfully turn into a geek we need to login for every test case unnecessarily considering that where're already inside the forms page.

To optimize it we'll update our scenarios to new variant scenarios:

1. First we'll create a new test suite AttemptBeGeek.robot that will contain our dynamic keyword

```
*** Keywords ***
Attempt Be a Geek
    [Arguments]    ${key}    ${input_field}    ${output_message}

    ${user}    Factory User    attempt_be_geek

    Fill Geek Form    ${user}[geek_profile]
    Submit Geek Form
    Alert Span Should Be    ${output_message}
```

and starter session

```
Start Session For Attempt Be Geek
    ${user}    Factory User    attempt_be_geek

    Start Session
    Do Login    ${user}
    Go To Geek Form
```

**Obs.:** This will then be used as test setup hook to initialise the test suite.

2. We now link it to our new updated attempt be geek factory

```
'attempt_be_geek': {
    'name': 'Dio',
    'lastname': 'Linux',
    'email': 'dio@linux.com',
    'password': 'pwd123',
    'geek_profile': {
        'whats': '11999999999',
        'desc': 'Instalo Distros Ubuntu, Debian, ElementaryOS, PopOS, Linux Mint, Kurumin, Mandrake, ConnectivaFedora, RedHat, CentOS, Slackware, Genton, Archlinux, Kubuntu, Xubuntu, Suze, Mandriva, Edubuntu, KateOSSabayon Linux, Manjaro Linux, BigLinux, ZorinOS, Unit',
        'printer_repair': 'N??o',
        'work': 'Ambos',
        'cost': '200'
    }
}
```

3. We'll then add a dictonary to our attempt be a geek function add the key and input_field arguments whom will change the values inside the factory to simulate user attempts to be a geek with invalid forms.

```
    Set To Dictionary    ${user}[geek_profile]    ${key}    ${input_field}
```

4. After all that we'll finally add our attempt test cases inside the suite using our pre-defined keywords as templateand only informing the test dough

```
*** Test Cases ***
Should Not be a Geek
    [Template]    Attempt Be a Geek

    desc    Formato o seu PC    A descri????o deve ter no minimo 80 caracteres
    desc    ${long_desc}    A descri????o deve ter no m??ximo 255 caracteres
    desc    ${EMPTY}    Informe a descri????o do seu trabalho
    whats    11    O Whatsapp deve ter 11 digitos contando com o DDD
    whats    999999999    O Whatsapp deve ter 11 digitos contando com o DDD
    whats    ${EMPTY}    O Whatsapp deve ter 11 digitos contando com o DDD
    cost    aaaa    Valor hora deve ser num??rico
    cost    aa12    Valor hora deve ser num??rico
    cost    &!*%??    Valor hora deve ser num??rico
    cost    ${EMPTY}    Valor hora deve ser num??rico
```

and updated the factory in our users seed function inside database.robot

```
Users Seed
    ${user}    Factory User    login
    Insert User    ${user}

    ${user2}    Factory User    be_geek
    Insert User    ${user2}

    ${user3}    Factory User    attempt_be_geek
    Insert User    ${user3}
```

**Obs.:** we created the `${long_desc}` variable to store a long description text, just for aesthetic reasons.

Using this improved format we'll now test all defined sad paths in the same window, one after other.

### Test template improvement

One problem of using the template in a test case level is that we'll now have several different scenarios (all of them with different inputs and outputs) but only one described test case, meaning that after running all those different scenarios it'll all be considered as an individual test case. To improve it, showing the test status for every of those scenarios we'll define a test template in a suite level.

1. First we'll define our template inside the settings section:

```
*** Settings ***
Documentation       Attempt BeGeek Test Suite

Resource            ../resources/Base.robot
Library             Users

Test Setup          Start Session For Attempt Be Geek
Test Template       Attempt Be a Geek
```

2. Now we define a specific test case for each of our previous variable scenarios:

```
*** Test Cases ***
Short desc    desc    Formato o seu PC    A descri????o deve ter no minimo 80 caracteres
Long desc    desc    ${long_desc}    A descri????o deve ter no m??ximo 255 caracteres
Empty desc    desc    ${EMPTY}    Informe a descri????o do seu trabalho
Whats only ddd    whats    11    O Whatsapp deve ter 11 digitos contando com o DDD
Whats only number    whats    999999999    O Whatsapp deve ter 11 digitos contando com o DDD
Empty whats    whats    ${EMPTY}    O Whatsapp deve ter 11 digitos contando com o DDD
Cost letters    cost    aaaa    Valor hora deve ser num??rico
Cost alpha    cost    aa12    Valor hora deve ser num??rico
Cost special    cost    &!*%??    Valor hora deve ser num??rico
Empty cost    cost    ${EMPTY}    Valor hora deve ser num??rico
```

3. After all that we'll change our test setup hook to a suite setup inside the settings section in order to run the test initiator only once for all the test suite.

```
*** Settings ***
Documentation       Attempt BeGeek Test Suite

Resource            ../resources/Base.robot
Library             Users

Suite Setup         Start Session For Attempt Be Geek
Test Template       Attempt Be a Geek
```

---

# Smoke test

Smoke test isn't a critical test but it allows us to find potencial issues before proceeding to new steps. This test is used for critical scenarios and its purpose is to verify application health before a release or delivery.

To create smoke tests we:

1. Define critical important scenarios (generally speaking, the inicial and/or basic flows of the application that gives access to all other scenarios)

- Register a new user
- Login
- Be a Geek

2. We then set a `[Tags]    smoke` to each of those scenarios

```
*** Test Cases ***
Register a new user
    [Tags]    smoke
    ...
```

3. Add the indication `-i smoke` to our console shortcut

```
robot -d ./logs -i smoke tests
```

4. Run the smoke test through console

We'll now have an idea about the health of the application to allow or not its evolution

---

# Screen Resolution

Always consider the screen resolution while creating and executing test cases. To optimize it you can consider the lower required resolution so all elements can be displayed and tested properly for diverent devices.

We define which resolution we want by adding the function `Set Viewport Size    1440    900` inside our test initiator

---

# Clearing HTML forms

While executing our tests inside /be-geek to register a new geek all forms information is being kept before the library actually inserts its input. Here this is not a problem because the factory overwrite the old data from the necessary fields, but the ideal scenario is to clear the forms for each new test case scenario.

In order to do this we have a JavaScript function that find and clear web applications forms data by class name `document.getElementsByClassName("be-geek-form")[0].reset();`. Although robot cannot read JavaScript so we'll add a keyword (Execute JavaScript) so robot can use the function properly.

1. We'll create a new action called Reset Geek Form

Used inside the course (OUTDATED)

```
Reset Geek Form
    Execute JavaScript    document.getElementsByClassName("be-geek-form")[0].reset();
```

Up to date alternative method

```
Reset Geek Form
    Evaluate JavaScript    css=.be-geek-form    document.getElementsByClassName("be-geek-form")[0].reset()
```

**Important:** Execute JavaScript method was outdated so I updated it to the Evaluate JavaScript method entering the selector and then the JS function.

2. Then we're going to call this new keyword inside the Fill Geek Form action

```
Fill Geek Form
    [Arguments]    ${geek_profile}

    Reset Geek Form

    Fill Text    id=whatsapp    ${geek_profile}[whats]
    Fill Text    id=desc    ${geek_profile}[desc]

    Select Options By    id=printer_repair    text    ${geek_profile}[printer_repair]
    Select Options By    id=work    text    ${geek_profile}[work]

    Fill Text    id=cost    ${geek_profile}[cost]
```

This way every time that a form is filled all previous data is cleared before entering the new data.

---

# Dealing with conditional elements (IFs)

Until this point we've created a bunch of test case considering all basic inputs but we now need to deal with the conditional selectors, for example:

> For the print_repair select element. While true the user is classified as supreme geek, while false the user is classified as a normal geek only so...
>
> - If print_repair is true or equal to "Sim" then user is a supreme geek;
> - If print_repair is false or equal to "N??o" then user is a normal geek;
> - If there's no option selected the span message "Por favor, informe se voc?? ?? um Geek Supremo" is displayed

1. We created 2 new test cases to test submiting empty select fields to our [Attempt Be Geek Suite](tests/AttemptBeGeek.robot):

```
*** Test Cases ***
...
No printer repair    printer_repair    ${EMPTY}    Por favor, informe se voc?? ?? um Geek Supremo
No word    work    ${EMPTY}    Por favor, selecione o modelo de trabalho
```

**OBS.:** using the `${EMPTY}` variable in here it's not recommended once it will also remove the standard option leaving and empty field, instead of the normal Select option that can lead to future bugs in the application.

In order to improve this empty field we'll define a conditional statement inside the [Fill Geek Form](resources/actions/GeekActions.robot) action:

```
    IF    '${geek_profile}[printer_repair]'
        Select Options By    id=printer_repair    text    ${geek_profile}[printer_repair]
    END

    IF    ${geek_profile}[work]
        Select Options By    id=work    text    ${geek_profile}[work]
    END
```

By doing it the select fields will be selected only if there's a valid option for it and if not this steps is not considered, we skip it to the next step.

---

# Challenge 1 - module PRO

## Automate 3 new test cases inside the login suite:

- Required email

Given that I an inside the login page
When the email field is empty
And I submit the login button
Then a required field message is displayed
"E-mail obrigat??rio"

- Required password

Given that I an inside the login page
When the password field is empty
And I submit the login button
Then a required field message is displayed
"Senha obrigat??rio"

- Required fields

Given that I an inside the login page
When both email and password fields are empty
And I submit the login button
Then a required field message is displayed for each field
"E-mail obrigat??rio" and "Senha obrigat??rio"

---

# API Testing

Here all backend testing and database connection will be analysed and validated though plataforms lile Postman, Insomnia and/or, as used here, Thunder Client (VS Code extention). The principle of API testing it to validate servers requirements linking it to the GUI's expected behaviour.

## Thunder Client

Thunder client is a VS Code extention used to manage and test server queries without the need of an external application like Postman and/or Insomnia.

**IMPORTANT:** Thunder Client execute requirements and validations but it cannot deal with data workload properly so it will be converted to robot as well for automated testing. Although Thunder Client can still be usefull and interesting for API Testing.

### Collections

All start by creating a collection like the "Getgeeks" created in here.

To create a collection:

1. Open the Collections tab;
2. Select the 3 lines buttom on the right of the filter field;
3. Select to create a New Collection or to Import an existing according to your needs.

**Obs.:** selecting the "..." button on the right of a collection will display it's possible options/actions.

### HTML Methods

All HTML requests use methods in order to understand what it should actually do. Some of these requests are:

- GET
- POST
- PUT
- DELETE

**Note:** all new created request come with GET as standart, change it to your needs.

### Status Codes

[Mozilla Web Docs - Status Code](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status?_gl=1*1uzgpt1*_ga*MTQyNjEyMjEwMi4xNjcxNDYzODUw*_ga_37GXT4VGQK*MTY3ODA3NjI5MC42NS4xLjE2NzgwNzcxNDYuMC4wLjA.)

1. Informative responses (100 - 199)
2. Success responses (200 - 299)
3. Redirectioning (300-399)
4. Client errors (400 - 499)
5. Server errors (500 - 599)

**Notes:**
200 - OK, everything was executed correctly
400 - Credentials and/or information do not exist o
401 - Credentials and/or information exists but is not authorized or is invalid

**OBS.:** devtools can be used in web applications in order to get all required requests information

### Tests

#### Validating token

Token validation is possible by verifying token's character count (in this case 140) and then using the function .lengh indicating the json token inside the tests tab from thunder client: `json.token.length` and the value of character it should receive (140 characters in our case)

In addition to it we can also validate if the token is expired or not by using `json.expires_in` with the value of days until it expires (10d for the application we're considering)

To call the elemts it's very simple we just need to look for it's name displayed inside the response, then consider the output type and describing the functions we want it to use after the . like in the where we're creating our own json queries.

#### Front-end x back-end

API validation is important eventhough the front-end already validate and blocks API requirements when the informations or inputs don't meet expectancies, this is important because eventhough that the application works fine, when sharing or delivering the API forward to a customer or client it must consider all succesfull and alternative scenarios validations to behave as expected.

## Automating API Tests with robot

The API tests created until now we're basically done manually throw Thunder Client, although it's possible to automate them using robot as well, in order to do it:

1. First of all we'll need to use a library that will allow us to send our HTTP requests from robot. In here we'll be using the library [HTTP RequestsLibrary (Python)](https://github.com/MarketSquare/robotframework-requests#readme)

**OBS.:** this can be found by filtering HTTP resources inside [Robot Framework webpage](https://robotframework.org/)

2. We'll then import the installed library to our [Sessions test file](../backend/users/tests/Sessions.robot) where we're going to define the automation.

```
*** Settings ***
Documentation       Session route test suite

Library    RequestsLibrary
```

3. We add our requests with all the body and header data from the API and also inform the path in it:

```
*** Test Cases ***
User session
  ${payload}    Create Dictionary    email=test2@email.com    password=pwd123
  ${header}    Create Dictionary    Content-Type=application/json

  POST    ${API_USERS}/sessions    data=${payload}    headers=${header}
```

4. After all we'll add our validations:

```
*** Test Cases ***
    ...

    Status Should Be    200    ${response}
```

**OBS.:** header variable won't be needed here because the Request Library have a function json that already converts information to json, eliminating the need to inform the header's content-type. Considering it we'll remore the header and change the data function for the new json one.

The new test case will be:

```
*** Test Cases ***
User session
    ${payload}    Create Dictionary    email=test2@email.com    password=pwd123

    ${response}    POST    ${API_USERS}/sessions    json=${payload}

    Status Should Be    200    ${response}
```

5. And all the other validations as well:

```
*** Test Cases ***
User session
    ${payload}    Create Dictionary    email=test2@email.com    password=pwd123

    ${response}    POST    ${API_USERS}/sessions    json=${payload}

    Status Should Be    200    ${response}

    ${size}    Get Length    ${response.json()}[token]
    ${expected_size}    Convert To Integer    140

    Should Be Equal    ${expected_size}    ${size}
    Should Be Equal    10d    ${response.json()}[expires_in]
```

**OBS.:** it's important to consider using ${expected_size} integer convertion once the Get Length function returns a integer number of characters and Should Be Equal function expects a string by default.

### Alternative scenarios (sad paths)

Request Library returns error whenever the request response differs from status 200 causing the execution to end before the actual validation, in order to allow the validation of alternative scenarios (that are intended to return errors) we need to add a new argument `expected_status=any` to the response:

```
${response}    POST    ${API_USERS}/sessions    json=${payload}    expected_status=any
```

This allows us to move on to the next steps of the test regardless of response status.

## Sessions.robot

To use the template we'll create variables to each of our test cases inputs. Here we're using the & in order to use each variable as a dictionary for our test cases

## Users.robot

While adding a new user we need to keep in mind that the user can only be registered once. This means that using the same payload won't work. One option to it is using faker to generate random payloads which causes massive information input inside the database, making it impracticable. Another option is to create Delorean that refresh the database before every execution as we did for the front-end, although as where dealing with API requests, there's an even better way to do it by defining DELETE requests to the API while executing it.

In order to do it:
1. First we execute the deletion manually to visualize the API Request
2. From the request we get the token that is generated while the user is created, it will give each created user one specific id
3. After all that we add new steps to our [User.robot](../backend/users/tests/Users.robot) suite to identify the token and execute the delete request

**OBS.:** it's possible to visualize the id analysing the encoder web token with [JWT](https://jwt.io/)

And then we add all that to our test case:
1. We define a new dictionary to fill the email and password with New User factory's data:

```
*** Test Cases ***
Add new user
    ${user}    Factory New User

    # Getting the user token
    ${payload}    Create Dictionary    email=${user}[email]    password=${user}[password]
    ${response}    POST Session    ${payload}
```

2. We get the token from API's body as json:

```
${token}    Set Variable    Bearer ${response.json()}[token]
```

3. Define what is the desired action/request to the API, validating it's response:

```
DELETE User    ${token}

${response}    POST User    ${user}
Status Should Be    201    ${response}
```

4. Once this scenario will only happen for existing users, we'll add our delete request to a condition statement allowing validation only if the user actually exists:

```
IF  "200" in "${response}"
    # We also need to inform the Bearer carrier as well
    ${token}    Set Variable    Bearer ${response.json()}[token]

    # Delete from /user
    DELETE User    ${token}
END
```

**OBS.:** this way once the user exists, validation will be executed, if not return the status code

### Validating ID

After considering the user token we now should have an id for each registered user, and then we should be able to validate it, although this id should be a positive number then we'll validate it as well:

1. First we've created a variable to store the received id from API's response:

```
${user_id}    Set Variable    ${response.json()}[id]
```

2. Then we added the condition that the id should be greater than 0:

```
Should Be True    ${user_id} > 0
```

**OBS.:** this is important to validade when considering the usage of those IDs in the application, negative and/or alphanumeric values for ID may cause problems while using it.

---

# Usefull terminal commands

- robot -l NONE tasks/Delorean.robot - runs the file without generating log. It's mostly used for tasks.
- robot --help - show all possible commands with detailed description
- robot -o NONE - runs the file without generating output
- robot -r NONE - runs the file witout generating report
- robot -i tags_name - runs only test cases that contains the defined tag
- robot -e tags_name - excludes tests that contains the defined tag
- chmod +x run.sh - gives file "run.sh" permition to be executed. Turns the file executable. **Works for linux based terminals**
- ./run.sh - runs a file inside the actual file path
- run.bat - runs terminal shortcuts **This works for MS-DOS based terminals**
- .lower() - python method to turn text onto lower case only
- fullPage - it's a browser library feature normally used for screenshots, expects true (to show the hole page) or false (to show only a piece of the page)
- robot -v BROWSER:chromium -v HEADLESS:true - define a variable and gives a value to it

## Git

git init - initialize the git repository
git status - return repository and files status
git add . - add all pending changes to the staged file
git commit -m 'message' - commit all staged changes and define a message to the commit
git pull - pull github updates to the local environment
git push - push local updates to the github environment

## Linux

find ./logs/pabot_results -type f -name "\*.png" - will find all files that have .png extension
cp $(find ./logs/pabot_results -type f -name "\*.png") ./logs/browser/screenshot - copy the list to the screenshot file

## chmod +x run.sh

files.sh only runs in linux compatible terminals, for terminals MS-DOS based we need to use a file.bat which will then need additional changes. **For example:**

Files path uses \ instead of /:

```
robot -l NONE -o NONE -r NONE tasks\Delorean.robot
robot -d ./logs tests\Signup.robot
```

And also a different way of calling it:

```
run.bat
```

## Request Library

## json=${variable} - convert the request content-type to json

# Important links

- [Web application](https://geeks-web-andre.fly.dev/signup)
- [ElephantSQL](https://api.elephantsql.com/console/51ccfaa2-d261-4503-b858-da3b75125790/browser?#)
- [QAcademy course](https://app.qacademy.io/area/produto/item/149046)
- [Faker Library](https://pypi.org/project/Faker/)
- [Typora md editor](https://typora.io/)
- [JSON Web Tokens](https://jwt.io/)

## Pabot

- [Pabot Parallel Executor](https://pabot.org/)

Is a parallel executor that runs several tests at once based on the number of available processors. It's important to note that using cloud environment only gives 1 available processor so it won't work for cloud environment.

### Pabot screenshot path improvement

The problem of Pabot is that the Browser and/or Selenium libraries cannot access the screenshot file path so the report won't display the screenshot images.

In order for the screenshots to be displayed correctly we'll rearrange the output files in a way that the browser lib can understand. To do it:

1. We added 4 new preparation steps for the run.sh terminal shortcut that will delete old files and create brand new ones

```
# delete the old browser file and create a brand new one
rm -rf ./logs/browser
mkdir ./logs/browser
mkdir ./logs/browser/screenshot

# list all type png files from pabot_results and copy it to the new screenshot file
cp $(find ./logs/pabot_results -type f -name "*.png") ./logs/browser/screenshot
```

2. We created a new faker [Utility generator](resources/Utility.py) that will create unique ids to each screenshot

```py
from faker import Faker
fake = Faker()


def screenshot_name():
    return fake.sha1()
```

3. And then added this generated id function to a new variable inside our test case closer, defining a new argument filename to set a name to each screenshot file that will receive the random generated id.

```
After Test
    ${shot_name}    Screenshot Name
    Take Screenshot    fullPage=true    filename=${shot_name}
```

By doing this we now refresh browsers lib screenshot file before each execution, saving all report screenshots inside a brand new file with unique identifiers, making it possible to visualize the screenshots inside the report even when using the pabot.

## Typora

[Typora](https://typora.io/) is a paid md files editor that allows easier ways to create readable and writable md notes

**Obs.:** i'm not using it in here because it's now paid
