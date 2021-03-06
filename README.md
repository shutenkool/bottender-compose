# bottender-compose

[![npm](https://img.shields.io/npm/v/bottender-compose.svg?style=flat-square)](https://www.npmjs.com/package/bottender-compose)
[![Build Status](https://github.com/Yoctol/bottender-compose/workflows/Node.js%20CI/badge.svg)](https://github.com/Yoctol/bottender-compose/actions?query=workflow%3ANode.js%20CI+branch%3Amaster)
[![coverage](https://codecov.io/gh/Yoctol/bottender-compose/branch/master/graph/badge.svg)](https://codecov.io/gh/Yoctol/bottender-compose)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> An utility library for [Bottender](https://github.com/Yoctol/bottender) and higher-order handlers

![](https://user-images.githubusercontent.com/3382565/45618583-d116bb00-baa8-11e8-9661-b36d8eb834c2.png)

## Installation

```sh
npm install bottender-compose
```

## API Reference

### TOC

- [Actions](#actions)
  - [`series()`](#series)
  - [`parallel()`](#parallel)
  - [`random()`](#random)
  - [`branch()`](#branch)
  - [`condition()`](#condition)
  - [`match()`](#match)
  - [`platform()`](#platform)
  - [`weight()`](#weight)
  - [`noop()`](#noop)
  - [`repeat()`](#repeat)
  - [`delay()`](#delay)
  - [`setDisplayName()`](#setdisplayName)
  - [`attachOptions()`](#attachoptions)
  - [Logger Methods](#logger-methods)
  - [Context Methods](#context-methods)
- [Predicates](#predicates)
  - [`isTextMatch()`](#istextmatch)
  - [`isPayloadMatch()`](#ispayloadmatch)
  - [`hasStateEqual()`](#hasstateequal)
  - [`not()`](#not)
  - [`and()`](#and)
  - [`or()`](#or)
  - [`alwaysTrue()`](#alwaystrue)
  - [`alwaysFalse()`](#alwaysfalse)
  - [Event Predicates](#event-predicates)

### Actions

### `series()`

Creates a function that executes methods in series.

```js
const { series, sendText } = require('bottender-compose');

module.exports = function App() {
  return series([
    sendText('1. First Item'),
    sendText('2. Second Item'),
    sendText('3. Third Item'),
  ]);
};
```

### `parallel()`

Creates a function that executes methods in parallel.

```js
const { parallel, sendText } = require('bottender-compose');

module.exports = function App() {
  return parallel([
    sendText('- You got one of Items'),
    sendText('- You got one of Items'),
    sendText('- You got one of Items'),
  ]);
};
```

### `random()`

Creates a function that executes one of method randomly.

```js
const { random, sendText } = require('bottender-compose');

module.exports = function App() {
  return random([
    sendText('You got a random item: A'),
    sendText('You got a random item: B'),
    sendText('You got a random item: C'),
  ]);
};
```

### `branch()`

Creates a function that will process either the `onTrue` or the `onFalse` function depending upon the result of the condition predicate.  
Furthermore, `branch` can be sued as curry function.

```js
const { branch, sendText } = require('bottender-compose');

module.exports = function App() {
  return branch(
    context => true,
    sendText('You are the lucky one.'),
    sendText('Too bad.')
  );
};

// curry function
const trueConditionBranch = branch(context => true);

module.exports = function App() {
  return trueConditionBranch(
    sendText('You are the lucky one.'), 
    sendText('Too bad.')
  );
};
```

Or you can executes function on `true` and do nothing when received `false`.

```js
branch(context => true, sendText('You are the lucky one.'));
```

`branch` works well with [predicates](#predicates).

### `condition()`

Creates a function that encapsulates `if/else`, `if/else`, ... logic.

```js
const { condition, sendText } = require('bottender-compose');

module.exports = function App() {
  return condition([
    [context => false, sendText('a')],
    [context => false, sendText('b')],
    [context => true, sendText('c')],
  ]);
};
```

`condition` works well with [predicates](#predicates).

### `match()`

Creates a function that encapsulates value matching logic.

```js
const { match, sendText } = require('bottender-compose');

module.exports = function App() {
  return match('a', [
    ['a', sendText('You got a A')],
    ['b', sendText('You got a B')],
    ['c', sendText('You got a C')],
  ]);
};
```

It accepts function with `context` argument:

```js
module.exports = function App() {
  return match(context => context.state.answer, [
    ['a', sendText('You got a A')],
    ['b', sendText('You got a B')],
    ['c', sendText('You got a C')],
  ]);
};

// curry function
const matchAnswer = match(context => context.state.answer);

module.exports = function App() {
  return matchAnswer([
    ['a', sendText('You got a A')],
    ['b', sendText('You got a B')],
    ['c', sendText('You got a C')],
  ]);
};
```

To assign default action, use `_` as pattern:

```js
const { _, match, sendText } = require('bottender-compose');

module.exports = function App() {
  return match('a', [
    ['a', sendText('You got a A')],
    ['b', sendText('You got a B')],
    ['c', sendText('You got a C')],
    [_, sendText('You got something')],
  ])
};
```

### `platform()`

Creates a function that will process function depending upon the platform context.

```js
const {
  platform,
  sendGenericTemplate,
  sendImagemap,
} = require('bottender-compose');

module.exports = function App() {
  return platform({
    messenger: sendGenericTemplate(...),
    line: sendImagemap(...),
  });
};
```

Or you can use `others` key to match other platforms:

```js
platform({
  messenger: sendGenericTemplate(...),
  line: sendImagemap(...),
  others: sendText('Unsupported.'),
});
```

### `weight()`

Creates a function that randomly executes one of method by its weight.

```js
const { weight, sendText } = require('bottender-compose');

module.exports = function App() {
  return weight([
    [0.2, sendText('20%')],
    [0.4, sendText('40%')],
    [0.4, sendText('40%')],
  ]);
};
```

### `noop()`

Creates a no-op function.

```js
const { branch, sendText, noop } = require('bottender-compose');

module.exports = function App() {
  return branch(
    context => false,
    sendText('You are the lucky one.'),
    noop() // do exactly nothing...
  );
};
```

### `repeat()`

Creates a function that executes the method repeatedly.  
Furthermore, `repeat` can be sued as curry function.

```js
const { repeat, sendText } = require('bottender-compose');

module.exports = function App() {
  return repeat(3, sendText('This will be sent 3 times.'));
};

// curry function
const repeatFiveTimes = repeat(5);

module.exports = function App() {
  return repeatFiveTimes(sendText('This will be sent 5 times.'))
};
```

### `delay()`

Creates a function that executes methods after a number of milliseconds.

```js
const { series, delay, sendText } = require('bottender-compose');

module.exports = function App() {
  return series([
    sendText('1. First Item'),
    delay(1000),
    sendText('2. Second Item'),
    delay(1000),
    sendText('3. Third Item'),
  ]);
};
```

### `setDisplayName()`

Assigns to the `displayName` property on the action.

```js
const { setDisplayName, sendText } = require('bottender-compose');

setDisplayName('sayHello', sendText('hello'));

// curry function
setDisplayName('sayHello')(sendText('hello'));
```

### `attachOptions()`

Attaches additional options to the action.

```js
const { attachOptions, sendText } = require('bottender-compose');

module.exports = function App() {
  return attachOptions(
    { tag: 'ISSUE_RESOLUTION' }, 
    sendText('Issue Resolved')
  );
};

// curry function
const attachIssueResolutionTag = attachOptions({ 
  tag: 'ISSUE_RESOLUTION',
});

module.exports = function App() {
  reutnr attachIssueResolutionTag(sendText('Issue Resolved'));
};
```

### Logger Methods

```js
B.series([
  B.log('sending hello'),
  B.info('sending hello'),
  B.warn('sending hello'),
  B.error('sending hello'),

  B.sendText('hello'),
]);
```

It supports template too.

```js
B.series([
  B.log('user: {{ session.user.id }} x: {{ state.x }}'),
  B.sendText('hello'),
]);
```

You can use your owner adapter for the logger:

```js
const { log, info, warn, error } = B.createLogger({
  log: debug('log'),
  info: debug('info'),
  warn: debug('warn'),
  error: debug('error'),
});

B.series([log('sending hello'), B.sendText('hello')]);
```

### Context Methods

#### Common

- `setState()`
- `resetState()`
- `typing()`

#### Messenger

- `sendMessage()`
- `sendText()`
- `sendAttachment()`
- `sendAudio()`
- `sendImage()`
- `sendVideo()`
- `sendFile()`
- `sendTemplate()`
- `sendButtonTemplate()`
- `sendGenericTemplate()`
- `sendListTemplate()`
- `sendOpenGraphTemplate()`
- `sendMediaTemplate()`
- `sendReceiptTemplate()`
- `sendAirlineBoardingPassTemplate()`
- `sendAirlineCheckinTemplate()`
- `sendAirlineItineraryTemplate()`
- `sendAirlineUpdateTemplate()`
- `sendSenderAction()`
- `markSeen()`
- `typingOn()`
- `typingOff()`
- `passThreadControl()`
- `passThreadControlToPageInbox()`
- `takeThreadControl()`
- `requestThreadControl()`
- `associateLabel()`
- `dissociateLabel()`

#### LINE

- `sendText()`
- `sendImage()`
- `sendVideo()`
- `sendAudio()`
- `sendLocation()`
- `sendSticker()`
- `sendImagemap()`
- `sendButtonTemplate()`
- `sendConfirmTemplate()`
- `sendCarouselTemplate()`
- `sendImageCarouselTemplate()`
- `reply()`
- `replyText()`
- `replyImage()`
- `replyVideo()`
- `replyAudio()`
- `replyLocation()`
- `replySticker()`
- `replyImagemap()`
- `replyButtonTemplate()`
- `replyConfirmTemplate()`
- `replyCarouselTemplate()`
- `replyImageCarouselTemplate()`
- `push()`
- `pushText()`
- `pushImage()`
- `pushVideo()`
- `pushAudio()`
- `pushLocation()`
- `pushSticker()`
- `pushImagemap()`
- `pushButtonTemplate()`
- `pushConfirmTemplate()`
- `pushCarouselTemplate()`
- `pushImageCarouselTemplate()`
- `linkRichMenu()`
- `unlinkRichMenu()`

#### Slack

- `sendText()`
- `postMessage()`
- `postEphemeral()`

#### Telegram

- `sendText()`
- `sendMessage()`
- `sendPhoto()`
- `sendAudio()`
- `sendDocument()`
- `sendSticker()`
- `sendVideo()`
- `sendVoice()`
- `sendVideoNote()`
- `sendMediaGroup()`
- `sendLocation()`
- `sendVenue()`
- `sendContact()`
- `sendChatAction()`
- `editMessageText()`
- `editMessageCaption()`
- `editMessageReplyMarkup()`
- `deleteMessage()`
- `editMessageLiveLocation()`
- `stopMessageLiveLocation()`
- `forwardMessageFrom()`
- `forwardMessageTo()`
- `kickChatMember()`
- `unbanChatMember()`
- `restrictChatMember()`
- `promoteChatMember()`
- `exportChatInviteLink()`
- `setChatPhoto()`
- `deleteChatPhoto()`
- `setChatTitle()`
- `setChatDescription()`
- `setChatStickerSet()`
- `deleteChatStickerSet()`
- `pinChatMessage()`
- `unpinChatMessage()`
- `leaveChat()`
- `sendInvoice()`
- `answerShippingQuery()`
- `answerPreCheckoutQuery()`
- `answerInlineQuery()`
- `sendGame()`
- `setGameScore()`

#### Viber

- `sendMessage()`
- `sendText()`
- `sendPicture()`
- `sendVideo()`
- `sendFile()`
- `sendContact()`
- `sendLocation()`
- `sendURL()`
- `sendSticker()`
- `sendCarouselContent()`

#### Facebook

- `sendComment()`
- `sendPrivateReply()`
- `sendLike()`

### Passing Function as Argument to Context Method

You can pass function as argument to handle time-specified or context-specified case, for example:

```js
// Lazy execution
B.sendText(() => `Now: ${new Date()}`);

// Use event information
B.sendText(context => `Received: ${context.event.text}`);
```

### Use Template in String

You can use `context`, `session`, `event`, `state` to access values in your template string:

```js
B.sendText('Received: {{event.text}}');
B.sendText('State: {{state.xxx}}');
```

Or use `props` to access object values that provided as props when calling action:

```js
B.sendText('User: {{props.name}}')(context, { name: 'Super User' });
```

### Predicates

### `isTextMatch()`

Creates a predicate function to return true when text matches.

```js
const { isTextMatch } = require('bottender-compose');

isTextMatch('abc')(context); // boolean
isTextMatch(/abc/)(context); // boolean
```

### `isPayloadMatch()`

Creates a predicate function to return true when payload matches.

```js
const { isPayloadMatch } = require('bottender-compose');

isPayloadMatch('abc')(context); // boolean
isPayloadMatch(/abc/)(context); // boolean
```

### `hasStateEqual()`

Creates a predicate function to return true when state matches.

```js
const { hasStateEqual } = require('bottender-compose');

hasStateEqual('x', 1)(context); // boolean
hasStateEqual('x.y.z', 1)(context); // boolean
hasStateEqual('x', { y: { z: 1 } })(context); // boolean
```

### `not()`

Creates a predicate function with **not** condition.

```js
const { not, hasStateEqual } = require('bottender-compose');

const predicate = not(hasStateEqual('x', 1));

predicate(context); // boolean
```

### `and()`

Creates a predicate function with **and** condition.

```js
const { and, hasStateEqual } = require('bottender-compose');

const predicate = and([
  isTextMatch('abc'),
  hasStateEqual('x', 1))
]);

predicate(context) // boolean
```

### `or()`

Creates a predicate function with **or** condition.

```js
const { or, hasStateEqual } = require('bottender-compose');

const predicate = or([
  isTextMatch('abc'),
  hasStateEqual('x', 1))
]);

predicate(context) // boolean
```

### `alwaysTrue`

Creates a predicate function that always return `true`.

```js
const { alwaysTrue } = require('bottender-compose');

const predicate = alwaysTrue();

predicate(context); // true
```

### `alwaysFalse`

Creates a predicate function that always return `false`.

```js
const { alwaysFalse } = require('bottender-compose');

const predicate = alwaysFalse();

predicate(context); // false
```

### Event Predicates

#### Messenger

- `isMessage()`
- `isText()`
- `hasAttachment()`
- `isImage()`
- `isAudio()`
- `isVideo()`
- `isLocation()`
- `isFile()`
- `isFallback()`
- `isSticker()`
- `isLikeSticker()`
- `isQuickReply()`
- `isEcho()`
- `isPostback()`
- `isGamePlay()`
- `isOptin()`
- `isPayment()`
- `isCheckoutUpdate()`
- `isPreCheckout()`
- `isRead()`
- `isDelivery()`
- `isPayload()`
- `isPolicyEnforcement()`
- `isAppRoles()`
- `isStandby()`
- `isPassThreadControl()`
- `isTakeThreadControl()`
- `isRequestThreadControl()`
- `isRequestThreadControlFromPageInbox()`
- `isFromCustomerChatPlugin()`
- `isReferral()`
- `isBrandedCamera()`

#### LINE

- `isMessage()`
- `isText()`
- `isImage()`
- `isVideo()`
- `isAudio()`
- `isLocation()`
- `isSticker()`
- `isFollow()`
- `isUnfollow()`
- `isJoin()`
- `isLeave()`
- `isPostback()`
- `isPayload()`
- `isBeacon()`
- `isAccountLink()`

#### Slack

- `isMessage()`
- `isChannelsMessage()`
- `isGroupsMessage()`
- `isImMessage()`
- `isMpimMessage()`
- `isText()`
- `isInteractiveMessage()`
- `isAppUninstalled()`
- `isChannelArchive()`
- `isChannelCreated()`
- `isChannelDeleted()`
- `isChannelHistoryChanged()`
- `isChannelRename()`
- `isChannelUnarchive()`
- `isDndUpdated()`
- `isDndUpdated_user()`
- `isEmailDomainChanged()`
- `isEmojiChanged()`
- `isFileChange()`
- `isFileCommentAdded()`
- `isFileCommentDeleted()`
- `isFileCommentEdited()`
- `isFileCreated()`
- `isFileDeleted()`
- `isFilePublic()`
- `isFileShared()`
- `isFileUnshared()`
- `isGridMigrationFinished()`
- `isGridMigrationStarted()`
- `isGroupArchive()`
- `isGroupClose()`
- `isGroupHistoryChanged()`
- `isGroupOpen()`
- `isGroupRename()`
- `isGroupUnarchive()`
- `isImClose()`
- `isImCreated()`
- `isImHistoryChanged()`
- `isImOpen()`
- `isLinkShared()`
- `isMemberJoinedChannel()`
- `isMemberLeftChannel()`
- `isPinAdded()`
- `isPinRemoved()`
- `isReactionAdded()`
- `isReactionRemoved()`
- `isStarAdded()`
- `isStarRemoved()`
- `isSubteamCreated()`
- `isSubteamMembersChanged()`
- `isSubteamSelfAdded()`
- `isSubteamSelfRemoved()`
- `isSubteamUpdated()`
- `isTeamDomainChange()`
- `isTeamJoin()`
- `isTeamRename()`
- `isTokensRevoked()`
- `isUrlVerification()`
- `isUserChange()`

#### Telegram

- `isMessage()`
- `isText()`
- `isAudio()`
- `isDocument()`
- `isGame()`
- `isPhoto()`
- `isSticker()`
- `isVideo()`
- `isVoice()`
- `isVideoNote()`
- `isContact()`
- `isLocation()`
- `isVenue()`
- `isEditedMessage()`
- `isChannelPost()`
- `isEditedChannelPost()`
- `isInlineQuery()`
- `isChosenInlineResult()`
- `isCallbackQuery()`
- `isPayload()`
- `isShippingQuery()`
- `isPreCheckoutQuery()`

#### Viber

- `isMessage()`
- `isText()`
- `isPicture()`
- `isVideo()`
- `isFile()`
- `isSticker()`
- `isContact()`
- `isURL()`
- `isLocation()`
- `isSubscribed()`
- `isUnsubscribed()`
- `isConversationStarted()`
- `isDelivered()`
- `isSeen()`
- `isFailed()`

#### Facebook

- `isFeed()`
- `isStatus()`
- `isStatusAdd()`
- `isStatusEdited()`
- `isPost()`
- `isPostRemove()`
- `isComment()`
- `isCommentAdd()`
- `isCommentEdited()`
- `isCommentRemove()`
- `isLike()`
- `isLikeAdd()`
- `isLikeRemove()`
- `isReaction()`
- `isReactionAdd()`
- `isReactionEdit()`
- `isReactionRemove()`

## License

MIT © [Yoctol](https://github.com/Yoctol/bottender-compose)
