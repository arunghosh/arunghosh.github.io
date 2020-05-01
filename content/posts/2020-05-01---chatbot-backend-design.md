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


Lately I was working on an ecommmerce web application. During the covid-19 lockdown there came a requirement, to create a chatbot that can 
1. Handle customer issues like
   - refund issue
   - issue with placing order 
   - other order related issues  
2. Assist shopping by helping find the products and offers

It came as a blessing since I can focus on something other than these convid based news and trolls.

From the requirements it was **rule-based** chatbot. It will be driven by the bot, i.e. the bot will ask some questions and provide the user with opions. And based on the option selected by the user the bot will decide on the subsequent message. So it is more of **Conversational User Interface(CUI)** than a fully fledged chatbot.

I had 2 options
1. Use an existing platforms like [Yellow Messenger](https://yellowmessenger.com/), [Dialogflow](https://dialogflow.com/) or
2. Create a custom chatbot from scratch

I choose the later. Why? Because it gives you
- More flexibility
- More learning oppourtunity
- More Challenge

This is Part-I of the series which will explain the basic backend design. The technology I choose were `ReactJS` and `NestJS` since I am comfortable with these.

## Conversation context/state

Now let us get into the basic steps in the chatbot conversation
1. The conversation is initiated by the bot.
2. The bot provides user with text and set of options.
3. User selects an option.
4. Bot processes user option and sends response.

In last step, where the bot process the user input, the application need `chat context/state`. This need to presisted in the server-side against the current user session. It is more like a user session.

```json
session_id => (user chat context/state)
```

For that each conversation will have a `session id` and the `state` will be stored against it.

The state should have
- User infomation like user ID and name.
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

This requires to have `current intent` and `intent stack`. So the intent infomation within the state will look like

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
This is similar to how call stack work. But call stack is much more complicated.

So now the state with user infomation and intent will look 
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
1. `prerequisites` an array of intents. These intents should be satisfied before you can execure the current intent. For example `refund_issue` can have prerequisite like `send_sorry` and `find_order_having_issue`.

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

A simple state handler to send sorry message will look like this

```typescript
const SORRY_MESSAGE: IIntentHandler = {
  prerequisites: [],
  updateState: (state, userInput) => handleAutoReply(state, userInput),
  getMessage: state => {
    return {
      texts: ["Sorry to hear that."],
    }
  }
}
```

A for a more complex use case like resolving the refund issue the handler will look like 

```typescript
const RESOLVE_REFUND_ISSUE: IIntentHandler = {
    prerequisites: [
        // First we need to send a sorry message to user 
        IntentType.SorryMessage,
        // Then find the order which is having the issue
        IntentType.SelectOrder,
    ],
  updateState: (state, userInput) => handleUserReply(state, userInput),
  getMessage: state => getMessage(state)
};
```

