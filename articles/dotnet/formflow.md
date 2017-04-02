---
title: FormFlow in the Bot Builder SDK for .NET | Microsoft Docs
description: Learn how to implement FormFlow with the Bot Builder SDK for .NET.
keywords: Bot Framework, .NET, Bot Builder, SDK, FormFlow
author: kbrandl
manager: rstand
ms.topic: develop-dotnet-article
ms.prod: botframework
ms.service: Bot Builder
ms.date: 03/28/2017
ms.reviewer:
#ROBOTS: Index
---

# FormFlow

[Dialogs](~/dotnet/dialogs.md) are very powerful and flexible, but handling a guided conversation such as ordering a sandwich can require a lot of effort. At each point in the conversation, there are many possibilities of what will happen next. For example, you may need to clarify an ambiguity, provide help, go back, or show progress. 
By using **FormFlow** within the Bot Builder SDK for .NET, you can greatly simplify the process of managing 
a guided conversation like this. 

FormFlow automatically generates the dialogs that are necessary to manage a guided conversation, 
based upon guidelines that you specify. 
Although using FormFlow sacrifices some of the flexibility that you might otherwise get by creating and managing 
dialogs on your own, designing a guided conversation using FormFlow can significantly reduce the time it takes 
to develop your bot. 
Additionally, you may construct your bot using a combination of FormFlow-generated dialogs and other types of 
dialogs. For example, a FormFlow dialog may guide the user through the process of completing a form, while a 
<a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/d8/df9/class_microsoft_1_1_bot_1_1_builder_1_1_dialogs_1_1_luis_dialog.html" target="_blank">LuisDialog</a> may evaluate user input to determine intent.

This article describes how to create a bot that uses the basic features of FormFlow to 
collect information from a user.

##<a id="forms-and-fields"></a>Forms and fields

To create a bot using FormFlow, you must specify the information that the bot needs to collect from the user. 
For example, if the bot's objective is to obtain a user's sandwich order, then you must define a form 
that contains fields for the data that the bot needs to fulfill the order. 
You can define the form by creating a C# class that contains one or more public properties to 
represent the data that the bot will collect from the user. 
Each property must be one of these data types:

- Integral (sbyte, byte, short, ushort, int, uint, long, ulong)
- Floating point (float, double)
- String
- DateTime
- Enumeration
- List of enumerations

Any of the data types may be nullable, which you can use to model that the field does not have a value. 
If a form field is based on an enumeration property that is not nullable, the value **0** in the enumeration represents **null** (i.e., indicates that the field does not have a value), 
and you should start your enumeration values at **1**. 
FormFlow ignores all other property types and methods.

For complex objects, you must create a form for the top-level C# class and another form for the complex object. 
You can compose the forms together by using typical [dialog](~/dotnet/dialogs.md) semantics. 
It is also possible to define a form directly by implementing [Advanced.IField](https://docs.botframework.com/en-us/csharp/builder/sdkreference/d5/d09/interface_microsoft_1_1_bot_1_1_builder_1_1_form_flow_1_1_advanced_1_1_i_field.html) or using [Advanced.Field](https://docs.botframework.com/en-us/csharp/builder/sdkreference/d2/d1b/class_microsoft_1_1_bot_1_1_builder_1_1_form_flow_1_1_advanced_1_1_field.html) and populating the dictionaries within it. 

> [!NOTE]
> You can define a form by using either a C# class or JSON schema. 
> This article describes how to define a form using a C# class. 
> For more information about using JSON schema, see [Define a form using JSON](~/dotnet/formflow-json-schema.md).

## Simple sandwich bot

Consider this example of a simple sandwich bot that is designed to obtain a user's sandwich order. 

###<a id="create-class"></a> Create the form

The `SandwichOrder` class defines the form and the enumerations define the options for building a sandwich. 
The class also includes the static `BuildForm` method that uses [FormBuilder](https://docs.botframework.com/en-us/csharp/builder/sdkreference/de/d9d/class_microsoft_1_1_bot_1_1_builder_1_1_form_flow_1_1_form_builder.html) to create the form and 
define a simple welcome message. 

To use FormFlow, you must first import the `Microsoft.Bot.Builder.FormFlow` namespace.

[!code-csharp[Define form](~/includes/code/dotnet-formflow.cs#defineForm)]

### Connect the form to the framework 

To connect the form to the framework, you must add it to the controller. 
In this example, the `Conversation.SendAsync` method calls the static `MakeRootDialog` method, 
which in turn, calls the `FormDialog.FromForm` method to create the `SandwichOrder` form. 

[!code-csharp[Connect form to framework](~/includes/code/dotnet-formflow.cs#connectToFramework)]

### See it in action

By simply defining the form with a C# class and connecting it to the framework, you have enabled FormFlow to 
automatically manage the conversation between bot and user. 
The example interactions shown below demonstrate the capabilities of a bot that is created by using the 
basic features of FormFlow. 
In each interaction, a **>** symbol indicates the point at which the user enters a response. 

#### Display the first prompt 

This form populates the `SandwichOrder.Sandwich` property. 
The form automatically generates the prompt, "Please select a sandwich", 
where the word "sandwich" in the prompt derives from the property name `Sandwich`. 
The `SandwichOptions` enumeration defines the choices that are presented to the user, 
with each enumeration value being automatically broken into words based upon changes in case and underscores.

```
Please select a sandwich
1. BLT
2. Black Forest Ham
3. Buffalo Chicken
4. Chicken And Bacon Ranch Melt
5. Cold Cut Combo
6. Meatball Marinara
7. Oven Roasted Chicken
8. Roast Beef
9. Rotisserie Style Chicken
10. Spicy Italian
11. Steak And Cheese
12. Sweet Onion Teriyaki
13. Tuna
14. Turkey Breast
15. Veggie
>
```

#### Provide guidance

The user can enter "help" at any point in the conversation to get guidance with filling out the form. 
For example, if the user enters "help" at the sandwich prompt, the bot will respond with this guidance. 

```
> help
* You are filling in the sandwich field. Possible responses:
* You can enter a number 1-15 or words from the descriptions. (BLT, Black Forest Ham, Buffalo Chicken, Chicken And Bacon Ranch Melt, Cold Cut Combo, Meatball Marinara, Oven Roasted Chicken, Roast Beef, Rotisserie Style Chicken, Spicy Italian, Steak And Cheese, Sweet Onion Teriyaki, Tuna, Turkey Breast, and Veggie)
* Back: Go back to the previous question.
* Help: Show the kinds of responses you can enter.
* Quit: Quit the form without completing it.
* Reset: Start over filling in the form. (With defaults from your previous entries.)
* Status: Show your progress in filling in the form so far.
* You can switch to another field by entering its name. (Sandwich, Length, Bread, Cheese, Toppings, and Sauce).
```

#### Advance to the next prompt

If the user enters "2" in response to the initial sandwich prompt, 
the bot then displays a prompt for the next property that is defined by the form: `SandwichOrder.Length`.

```
Please select a sandwich
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> 2
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### Return to the previous prompt 

If the user enters "back" at this point in the conversation, the bot will return the previous prompt. 
The prompt shows the user's current choice ("Black Forest Ham"); the user may change that selection by 
entering a different number or confirm that selection by entering "c".

```
> back
Please select a sandwich(current choice: Black Forest Ham)
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> c
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### Clarify user input

If the user responds with text (instead of a number) to indicate a choice, 
the bot will automatically ask for clarification if user input matches more than one choice. 

```
Please select a bread
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> nine grain
By "nine grain" bread did you mean (1. Nine Grain Honey Oat, 2. Nine Grain Wheat)
> 1
```

If user input does not directly match any of the valid choices, the bot will 
automatically prompt the user for clarification.

```
Please select a cheese (1. American, 2. Monterey Cheddar, 3. Pepperjack)
> amercan
"amercan" is not a cheese option.
> american smoked
For cheese I understood American. "smoked" is not an option.
```

If user input specifies multiple choices for a property and the bot does not understand any of the 
specified choices, it will automatically prompt the user for clarification.

```
Please select one or more toppings
 1. Banana Peppers
 2. Cucumbers
 3. Green Bell Peppers
 4. Jalapenos
 5. Lettuce
 6. Olives
 7. Pickles
 8. Red Onion
 9. Spinach
 10. Tomatoes
> peppers, lettuce and tomato
By "peppers" toppings did you mean (1. Green Bell Peppers, 2. Banana Peppers)
> 1
```

#### Show current status

If the user enters "status" at any point in the order, the bot's response will indicate 
which values have already been specified and which values remain to be specified. 

```
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> status
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Unspecified  
```

#### Confirm selections

When the user completes the form, the bot will ask the user to confirm their selections.

```
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> 1
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
>
```

If the user responds by entering "no", the bot allows the user to update any of the prior selections. 
If the user responds by entering "yes", the form has been completed and control is returned to the calling dialog. 

```
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> no
What do you want to change?
 1. Sandwich(Black Forest Ham)
 2. Length(Six Inch)
 3. Bread(Nine Grain Honey Oat)
 4. Cheese(American)
 5. Toppings(Lettuce, Tomatoes, and Green Bell Peppers)
 6. Sauce(Honey Mustard)
> 2
Please select a length (current choice: Six Inch) (1. Six Inch, 2. Foot Long)
> 2
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Foot Long
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> y
```

## Handling quit and exceptions

If the user enters "quit" in the form or an exception occurs at some point in the conversation, 
your bot will need to know the step in which the event occurred, the state of the form when the event occurred, 
and which steps of the form were successfully completed prior to the event. 
The form returns this information via the `FormCanceledException<T>` class. 

This code example shows how to catch the exception and display a message according to the event that occurred. 

[!code-csharp[Handle exception or quit](~/includes/code/dotnet-formflow.cs#handleExceptionOrQuit)]

## Summary

This article has described how to use the basic features of FormFlow to create a bot that can:

- Automatically generate and manage the conversation
- Provide clear guidance and help
- Understand both numbers and textual entries
- Provide feedback to the user regarding what is understood and what is not 
- Ask clarifying questions when necessary 
- Allow the user to navigate between steps

Although basic FormFlow functionality is sufficient in some cases, 
you should consider the potential benefits of incorporating some of the more advanced 
features of FormFlow into your bot. 
For more information, see [Advanced features of FormFlow](~/dotnet/formflow-advanced.md) and [Customize a form using FormBuilder](~/dotnet/formflow-formbuilder.md).

## Additional resources

- [Advanced features of FormFlow](~/dotnet/formflow-advanced.md)
- [Customize a form using FormBuilder](~/dotnet/formflow-formbuilder.md)
- [Localize form content](~/dotnet/formflow-localize.md)
- [Define a form using JSON schema](~/dotnet/formflow-json-schema.md)
- [Customize user experience with pattern language](~/dotnet/formflow-pattern-language.md)
