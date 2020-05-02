---
title: Rule-based chatbot from scratch using ReactJS and NodeJS
date: "2020-04-30T22:40:32.169Z"
template: "post"
draft: false
slug: "rule-based-chatbot-from-scratch-using-reactjs-and-nodejs-p1"
category: "Chatbot"
tags:
  - "Chatbot"
  - "Conversation UI"
description: "This gives you pointers on how to create a rule-based chatbot from scratch"
---


During [Covid-19 lockdown in India](https://en.wikipedia.org/wiki/2020_coronavirus_lockdown_in_India), I was working on an e-commerce web application. The work was mostly maintenance, which was hard to keep you interested during a time when you are surrounded by the pandemic news.

As a saviour there came a requirement. It was to create a chatbot that can
1. Handle customer issues like
   - refund issue
   - issue with placing order 
   - other order related issues  
2. Assist shopping by helping find the products and offers

It came as a blessing since I can shift my focus on something other than the news and trolls.

From the requirements it was a **rule-based** chatbot. The conversation will be driven by the bot, i.e. the bot will ask some questions and provide the user with options. User can select and options and based on the option selected the bot will decide on the subsequent message. So it is more of **Conversational User Interface(CUI)** than a fully fledged chatbot.

To realise the requirement I had 2 options
1. Use an existing platforms like [Yellow Messenger](https://yellowmessenger.com/), [Dialogflow](https://dialogflow.com/) or
2. Create a custom chatbot from scratch

I choose the later. Why? Because it gives you
- More flexibility
- More learning opportunity
- More Challenge

This article will explain the basic backend design of the chatbot. The technology I choose was `ReactJS` and `NestJS`. The main reason being, I am comfortable with these and there is no need to switch context since both are JavaScript.

## Conversation context/state

Now let us get into the basic steps in the chatbot conversation
1. The conversation is initiated by the bot.
2. The bot provides user with text and set of options.
3. User selects an option.
4. Bot processes user option and sends response.

In last step, where the bot process the user input, the application need `chat context/state`. This need to be persisted in the server-side against the current user session. It is more like a user session.

```json
session_id => (user chat context/state)
```

For that each conversation will have a `session id` and the `state` will be stored against it.

The state should have
- User information like user ID and name.
- What is the aim/intent of the conversation. For example initially the intent of the bot is to find what help the user need. If the user selects "refund issue", then the intent is to resolve refund issue.
 
The state should have all the information required for the conversation. And so as of now we know that the session will have details on the user and intent.

```typescript
interface IState {
  user: {
    session: string;
    id: string;
    name?: string;
  };
  intent: {
    current: IntentType;
    stack: IntentType[];
  };
}

```

## Intent

Since the conversation is driven by the bot, at any point of the conversation the bot will have an `intent`. For example resolve refund issue, provide offer details, welcome user.

An intent can have many steps. For example, an solve refund issue bot need to:
1. Send an apology for the issue.
2. Find the order having the issue.
3. Find analyze situation and send a response.

This requires to have `current intent` and `intent stack`. So the intent information within the state will look like

```typescript
{
    intent: {
        // Current intent
        // Ex: Resolve Refund Issue
        current: "resolve_refund_issue";
        // The intent stack
        stack: [];
    }
}
```
So what is this current and stack? Let's take an example ~ if the intent is the resolve refund issue, first we will be sending a sorry message.

Initially when you send the intent as `resolve_refund_issue` the state will look like:
``` json
current: "resolve_refund_issue",
stack: [],
```

Now the prerequisite of the intent will look like 
```json
prerequisite: ["send_sorry", "select_order"]
```

 So first we need to send the sorry message. To do this we stack the current intent and sent the current intent to `send_sorry`.
``` json
current: "send_sorry",
stack: ["resolve_refund_issue"],
```
It is like calling one function from another function. So similar to how call stack work. But the call stack is much more complicated.

So now the state with user information and intent will look 
```typescript
interface IState {
  user: {
    session: string;
    id: string;
  };
  intent: {
    current: IntentType;
    stack: IntentType[];
  };
}
```


## Intent handler

Each intent will have a handler associated with it. Intent handler will have following 

```typescript
interface IIntentHandler {
  // the prerequisites for the handler to work. For example the refund issue requires to send a sorry message and select an order
  prerequisites: IntentType[];
  // updateState takes in the state and user input and returns the updated state
  updateState: (arg0: IState, arg1: string) => Promise<IState>;
  // getMessage takes in state and provides message for the user
  getMessage: (arg0: IState) => IMessage;;
}
```
The **`prerequisites`** is an array of intents. These intents should be satisfied before you can execute the current intent. For example `refund_issue` can have prerequisite like `send_sorry` and `find_order_having_issue`. So as mentioned in the previous section when you need to execute `send_sorry`, you will make this the current intent and stack `refund_issue`. It is like calling one function from another function.

The **`updateState`** function takes in the current state and the user input and updates the state based on these two parameters. For example `updateState` for `select_order` intent will 
1. First update the state with the orders placed by the users for the purpose of asking user to select order. 
2. When the user selects the order, `updateState` shall update the state with the selected order.

The state can have an `execute` and `heap` object to store the details on the execution, like list of orders, selected orders & status of execution.

```typescript
{
  execute_status: {
      select_order: {
      result: any,
      success: boolean,
      completed: boolean, 
    }
  },
  heap: {
    select_order: {
      orders: object[] 
    }
  }
}
```

The **`getMessage`** function will return the message for the handler. The message will have text and options
```typescript
interface IMessage {
  texts: string[];
  options?: {
    text: string,
    id: string,
  };
}
```

A simple `getMessage` that requests user to select a brand will look like.
```javascript
getMessage: state => {
  return {
    texts: ['Which brand do you wish to explore?'],
    options: [
      {
        text: 'Wrogn Mens fashion by Virat Kholi',
        id: 'wrogn',
      },
      {
        text: 'HRX Mens fashion by Hrithik Roshan',
        id: 'hrx',
      },
    ],
  };
```

The above example since a simple case is not consuming the state. But there are usecases where in we need to use state to return appropriate message to the user.