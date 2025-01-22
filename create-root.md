# Learn the React deeply - createRoot

## Dependencies tree

関数の呼び出し、依存ツリー

* [createRoot](https://github.com/facebook/react/blob/v19.0.0/packages/react-dom/src/client/ReactDOMRoot.js#L161)
  * [createContainer](https://github.com/facebook/react/blob/v19.0.0/packages/react-reconciler/src/ReactFiberReconciler.js#L228)
    * [createFiberRoot](https://github.com/facebook/react/blob/v19.0.0/packages/react-reconciler/src/ReactFiberRoot.js#L144)
      * new FiberRootNode
      * [createHostRootFiber](https://github.com/facebook/react/blob/v19.0.0/packages/react-reconciler/src/ReactFiber.js#L527)
        * createFiber(createFiberImplClass)
      * initializeUpdateQueue
  * [markContainerAsRoot](https://github.com/facebook/react/blob/v19.0.0/packages/react-dom-bindings/src/client/ReactDOMComponentTree.js#L66)
  * [listenToAllSupportedEvents](https://github.com/facebook/react/blob/v19.0.0/packages/react-dom-bindings/src/events/DOMPluginEventSystem.js#L415)
    * listenToNativeEvent
      * addTrappedEventListener
        * [createEventListenerWrapperWithPriority](https://github.com/facebook/react/blob/v19.0.0/packages/react-dom-bindings/src/events/ReactDOMEventListener.js#L85)
          * getEventPriority
          * dispatchDiscreteEvent(dispatchEvent)
            * findInstanceBlockingEvent
            * [dispatchEventForPluginEventSystem](https://github.com/facebook/react/blob/v19.0.0/packages/react-dom-bindings/src/events/DOMPluginEventSystem.js#L566)
              * dispatchEventsForPlugins
                * getEventTarget
                * extractEvents
                  * [SimpleEventPlugin.extractEvents](https://github.com/facebook/react/blob/v19.0.0/packages/react-dom-bindings/src/events/plugins/SimpleEventPlugin.js#L56)
                    * [accumulateSinglePhaseListeners](https://github.com/facebook/react/blob/v19.0.0/packages/react-dom-bindings/src/events/DOMPluginEventSystem.js#L694)
                      * [getListener](https://github.com/facebook/react/blob/v19.0.0/packages/react-native-renderer/src/legacy-events/ResponderEventPlugin.js#L241)
                      * createDispatchListener
                    * [new SyntheticEventCtor(SyntheticEvent)](https://github.com/facebook/react/blob/v19.0.0/packages/react-dom-bindings/src/events/SyntheticEvent.js#L46)
                * processDispatchQueue
                  * processDispatchQueueItemsInOrder
                    * executeDispatch
  * new ReactDOMRoot

## The work of each function

### createRoot

1. createContainer を実行し root（FiberRoot）を返す。
2. markContainerAsRoot で root.current（HostRoot）を container["__reactContainer$"]へ代入する。
    *  root.current には createContainer の内部処理で HostRoot（Fiber）が代入されている。
3. listenToAllSupportedEvents の実行により rootContainerElement へ全ての event（"click", "touch", etc...）を addEventListener により登録する。イベントに対する callback 関数が実行された場合には React コンポーネントに登録された onClick, onTouchStart, onMouseOver, ...etc がイベントの発火に応じて実行される。

<details>

<summary>code</summary>

```ts
export function createRoot(
  container: Element | Document | DocumentFragment,
  options?: CreateRootOptions,
): RootType {
  // ...略

  // FiberRoot
  const root = createContainer(
    container,           // HTMLElement
    ConcurrentRoot,      // ConcurrentRoot = 1
    null,
    isStrictMode,
    concurrentUpdatesByDefaultOverride, // false
    identifierPrefix,    // option
    onUncaughtError,     // option
    onCaughtError,       // option
    onRecoverableError,  // option
    transitionCallbacks, // option
  );
  
  // root.current は HostRoot(Fiber)
  // container["__reactContainer$"] に HostRoot を代入する
  markContainerAsRoot(root.current, container);

  const rootContainerElement: Document | Element | DocumentFragment =
    container.nodeType === COMMENT_NODE
      ? (container.parentNode: any)
      : container;

  // rootContainerElement に全ての event を listen させる
  listenToAllSupportedEvents(rootContainerElement);

  return new ReactDOMRoot(root);
}
```

</details>

### createContainer

* 関数の実態は createFiberRoot

<details>

<summary>code</summary>

```ts
export function createContainer(
  containerInfo: Container,
  tag: RootTag,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
  isStrictMode: boolean,
  concurrentUpdatesByDefaultOverride: null | boolean,
  identifierPrefix: string,
  onUncaughtError: () => void,
  onCaughtError: () => void,
  onRecoverableError: () => void,
  transitionCallbacks: null | TransitionTracingCallbacks,
): OpaqueRoot {
  const hydrate = false;
  const initialChildren = null;
  return createFiberRoot(
    containerInfo,        // HTMLElement
    tag,                  // ConcurrentRoot = 1
    hydrate,              // false
    initialChildren,      // null
    hydrationCallbacks,   // null
    isStrictMode,         // true
    identifierPrefix,     // option
    onUncaughtError,      // option
    onCaughtError,        // option
    onRecoverableError,   // option
    transitionCallbacks,  // option
    null,
  );
}
```

</details>

### createContainer > createFiberRoot

1. new FiberRoot で root(FiberRoot) を作る
2. createHostRootFiber 実行で HostRoot(Fiber)
3. root.current に HostRoot を代入する
4. HostRoot.stateNode に root を代入する
5. HostRoot.memoizedState に state（RootState）を代入する
6. initializeUpdateQueue 実行で HostRoot.updateQueue へ値を代入
7. root（FiberRoot）を返す

<details>

<summary>code</summary>

```ts
export function createFiberRoot(
  containerInfo: Container,
  tag: RootTag,
  hydrate: boolean,
  initialChildren: ReactNodeList,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
  isStrictMode: boolean,
  identifierPrefix: string,
  onUncaughtError: () => void,
  onCaughtError: () => void,
  onRecoverableError: () => void,
  transitionCallbacks: null | TransitionTracingCallbacks,
  formState: ReactFormState<any, any> | null,
): FiberRoot {
  // FiberRoot を作成
  const root: FiberRoot = (new FiberRootNode(
    containerInfo, 
    tag,
    hydrate,
    identifierPrefix,
    onUncaughtError,
    onCaughtError,
    onRecoverableError,
    formState,
  ): any);

  // Fiber（HostRoot）を作成
  const uninitializedFiber = createHostRootFiber(tag, isStrictMode);

  // FiberRoot の current に Fiber（HostRoot）を設定
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;

  if (enableCache) {
    const initialCache = createCache();
    retainCache(initialCache);
    root.pooledCache = initialCache;
    retainCache(initialCache);

    const initialState: RootState = {
      element: initialChildren,
      isDehydrated: hydrate,
      cache: initialCache,
    };
    uninitializedFiber.memoizedState = initialState;
  } else {
    const initialState: RootState = {
      element: initialChildren,
      isDehydrated: hydrate,
      cache: (null: any), // not enabled yet
    };
    uninitializedFiber.memoizedState = initialState;
  }

  // HostRoot.updateQueue に値を代入
  initializeUpdateQueue(uninitializedFiber);

  return root;
}
```

</details>

### markContainerAsRoot

* container["__reactContainer$"] に HostRoot（Fiber）を代入する

```ts
export function markContainerAsRoot(hostRoot: Fiber, node: Container): void {
  node[internalContainerInstanceKey] = hostRoot;
}
```

### listenToAllSupportedEvents

* allNativeEvents を forEach でループして listenToNativeEvent を実行している
  * allNativeEvents は命名の通り "click","touchstart","mouseover" 等の標準的なイベント名の配列

<details>

<summary>code</summary>

```ts
const listeningMarker = '_reactListening' + Math.random().toString(36).slice(2);

export function listenToAllSupportedEvents(rootContainerElement: EventTarget) {
  if (!(rootContainerElement: any)[listeningMarker]) {
    (rootContainerElement: any)[listeningMarker] = true;
    allNativeEvents.forEach(domEventName => {
      // We handle selectionchange separately because it
      // doesn't bubble and needs to be on the document.
      if (domEventName !== 'selectionchange') {
        if (!nonDelegatedEvents.has(domEventName)) {
          // click, touch などはこっち
          listenToNativeEvent(domEventName, false, rootContainerElement);
        }
        // play, canPlay などはこっち
        listenToNativeEvent(domEventName, true, rootContainerElement);
      }
    });
    // .....
  }
}
```

</details>

### listenToAllSupportedEvents > listenToNativeEvent

1. 引数で受け取る isCapturePhaseListener の真偽値に応じて変数 eventSystemFlags に 0 or 4 を代入する
2. addTrappedEventListener 関数を実行


<details>

<summary>code</summary>

```ts
export function listenToNativeEvent(
  domEventName: DOMEventName,
  isCapturePhaseListener: boolean,
  target: EventTarget,
): void {
  // ...

  let eventSystemFlags = 0;
  if (isCapturePhaseListener) {
    // 4 が代入される
    eventSystemFlags |= IS_CAPTURE_PHASE;
  }
  addTrappedEventListener(
    target,                 // HTMLElement(root)
    domEventName,           // eventName
    eventSystemFlags,       // 0
    isCapturePhaseListener, // false
  );
}
```

</details>

### listenToAllSupportedEvents > listenToNativeEvent > addTrappedEventListener

1. createEventListenerWrapperWithPriority 関数の実行によりイベント（domEventName）に対応する listener 関数を取得する。
2. addEventBubbleListener により targetContainer にイベントに対する listener 関数を登録する。

<details>

<summary>code</summary>

```ts
function addTrappedEventListener(
  targetContainer: EventTarget,                     // HTMLElement
  domEventName: DOMEventName,                       // eventName
  eventSystemFlags: EventSystemFlags,               // 0
  isCapturePhaseListener: boolean,                  // false
  isDeferredListenerForLegacyFBSupport?: boolean,   // undefined
) {
  let listener = createEventListenerWrapperWithPriority(
    targetContainer,
    domEventName,
    eventSystemFlags,
  );
  
  //...

  targetContainer =
    // false -> targetContainer がそのまま代入される
    enableLegacyFBSupport && isDeferredListenerForLegacyFBSupport
      ? (targetContainer: any).ownerDocument
      : targetContainer;

  let unsubscribeListener;
  
  // ...

  if (isCapturePhaseListener) {
    if (isPassiveListener !== undefined) {
      unsubscribeListener = addEventCaptureListenerWithPassiveFlag(
        targetContainer,
        domEventName,
        listener,
        isPassiveListener,
      );
    } else {
      unsubscribeListener = addEventCaptureListener(
        targetContainer,
        domEventName,
        listener,
      );
    }
  } else {
    if (isPassiveListener !== undefined) {
      unsubscribeListener = addEventBubbleListenerWithPassiveFlag(
        targetContainer,
        domEventName,
        listener,
        isPassiveListener,
      );
    } else {
      // click event はここを通りそう
      unsubscribeListener = addEventBubbleListener(
        targetContainer,
        domEventName,
        listener,
      );
    }
  }
}
```

</details>

### listenToAllSupportedEvents > listenToNativeEvent > addTrappedEventListener > createEventListenerWrapperWithPriority

1. 引数で受け取ったイベント名に応じて getEventPriority 関数が優先度（文字列）を返す。
2. 優先度に応じて dispatchDiscreteEvent, dispatchContinuousEvent, dispatchEvent のいずれかの関数を bind して返す。
    * 内部処理的には dispatchDiscreteEvent, dispatchContinuousEvent 関数のどちらも dispatchEvent 関数を最終的に実行する。なので実態は殆ど dispatchEvent 関数である。
    * "click" イベントの場合は dispatchDiscreteEvent 関数が返される。
    * bind で返す関数は既に domEventName, eventSystemFlags, targetContainer が渡されているので、残りの event object（nativeEvent）を引数で受け取る関数（dispatchEvent）が返る。

<details>

<summary>code</summary>

```ts
export function createEventListenerWrapperWithPriority(
  targetContainer: EventTarget,
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
): Function {
  const eventPriority = getEventPriority(domEventName);
  let listenerWrapper;
  switch (eventPriority) {
    case DiscreteEventPriority:
      // click event など多くのイベントはこの case
      listenerWrapper = dispatchDiscreteEvent;
      break;
    case ContinuousEventPriority:
      listenerWrapper = dispatchContinuousEvent;
      break;
    case DefaultEventPriority:
    default:
      listenerWrapper = dispatchEvent;
      break;
  }
  return listenerWrapper.bind(
    null,
    domEventName,
    eventSystemFlags,
    targetContainer,
  );
}
```

</details>

### listenToAllSupportedEvents ~ addTrappedEventListener > createEventListenerWrapperWithPriority > dispatchEvent

return_targetInst にイベント発生元の target(Node) の Fiber が入り、dispatchEvent の実態は殆ど dispatchEventForPluginEventSystem である。

1. findInstanceBlockingEvent を実行
    1. 引数で受け取った nativeEvent から target(Node) を取り出す
    2. target から fiber を取得し return_targetInst にその fiber を代入する
        * return_targetInst は dispatchEvent 関数等が定義されているファイル内においてグローバルな変数
    3. target の fiber が suspense component 等でない限りは null を返す
2. dispatchEventForPluginEventSystem を実行

<details>

<summary>code</summary>

```ts
export function dispatchEvent(
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  targetContainer: EventTarget,
  nativeEvent: AnyNativeEvent,
): void {
  if (!_enabled) {
    return;
  }

  let blockedOn = findInstanceBlockingEvent(nativeEvent);
  if (blockedOn === null) {
    dispatchEventForPluginEventSystem(
      domEventName,       // eventName
      eventSystemFlags,   // 0
      nativeEvent,        // Event Object
      return_targetInst,  // Fiber(event target)
      targetContainer,    // HTMLElement
    );
    clearIfContinuousEvent(domEventName, nativeEvent);
    return;
  }
```

</details>

### listenToAllSupportedEvents ~ createEventListenerWrapperWithPriority > dispatchEvent > dispatchEventForPluginEventSystem

batchedUpdates 関数の callback として dispatchEventsForPlugins 関数を渡して実行している。batchedUpdates 関数は恐らく state 更新された場合に再レンダリングを纏めるためのもの？とりあえず dispatchEventForPluginEventSystem の実態は 殆ど dispatchEventsForPlugins 関数と言える。

<details>

<summary>code</summary>

```ts
export function dispatchEventForPluginEventSystem(
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  nativeEvent: AnyNativeEvent,
  targetInst: null | Fiber,
  targetContainer: EventTarget,
): void {
  let ancestorInst = targetInst;

  // ...略

  batchedUpdates(() =>
    dispatchEventsForPlugins(
      domEventName,
      eventSystemFlags,
      nativeEvent,
      ancestorInst,
      targetContainer,
    ),
  );
}

```

</details>

### listenToAllSupportedEvents ~ dispatchEvent > dispatchEventForPluginEventSystem > dispatchEventsForPlugins

1. dispatchQueue 配列を用意する
2. extractEvents が実行される
    * extractEvents 実行の過程でイベント発生元及び、その親コンポーネントで定義されている onXXXX（event listener）関数を含んだ配列が dispatchQueue の配列に push される。
3. processDispatchQueue 関数の実行で dispatchQueue 配列内の onXXXX 関数を取り出して実行する。onXXXX はイベント発生元で定義された関数だけでなく、親で定義された同様のイベントリスナー関数も for 文のループ内で実行されるため、このような仕組みで純粋なイベントのバブリングに伴うイベントリスナの実行を再現している。

<details>

<summary>code</summary>

```ts
function dispatchEventsForPlugins(
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  nativeEvent: AnyNativeEvent,
  targetInst: null | Fiber,
  targetContainer: EventTarget,
): void {
  // event object から target(Node) を取得
  const nativeEventTarget = getEventTarget(nativeEvent);
  const dispatchQueue: DispatchQueue = [];
  extractEvents(
    dispatchQueue,
    domEventName,
    targetInst,
    nativeEvent,
    nativeEventTarget,
    eventSystemFlags,
    targetContainer,
  );
  processDispatchQueue(dispatchQueue, eventSystemFlags);
}
```

</details>

### listenToAllSupportedEvents ~ dispatchEventForPluginEventSystem > dispatchEventsForPlugins > extractEvents

発生したイベントに応じて適切な XXXEventPlugin オブジェクトの extractEvents 関数を実行する。

<details>

<summary>code</summary>

```ts
function extractEvents(
  dispatchQueue: DispatchQueue,
  domEventName: DOMEventName,
  targetInst: null | Fiber,
  nativeEvent: AnyNativeEvent,
  nativeEventTarget: null | EventTarget,
  eventSystemFlags: EventSystemFlags,
  targetContainer: EventTarget,
) {
  // 多くは SimpleEventPlugin の extractEvents が実行される
  SimpleEventPlugin.extractEvents(
    dispatchQueue,
    domEventName,
    targetInst,
    nativeEvent,
    nativeEventTarget,
    eventSystemFlags,
    targetContainer,
  );
  const shouldProcessPolyfillPlugins =
    (eventSystemFlags & SHOULD_NOT_PROCESS_POLYFILL_EVENT_PLUGINS) === 0;

  if (shouldProcessPolyfillPlugins) {
    EnterLeaveEventPlugin.extractEvents(
      // ...
    );
    ChangeEventPlugin.extractEvents(
      // ...
    );
    SelectEventPlugin.extractEvents(
      // ...
    );
    BeforeInputEventPlugin.extractEvents(
      // ...
    );
    FormActionEventPlugin.extractEvents(
      // ...
    );
  }
}
```

</details>

### listenToAllSupportedEvents ~  dispatchEventsForPlugins > extractEvents > SimpleEventPlugin.extractEvents

1. 発生したイベントに応じて event listener 関数に渡す event オブジェクトを決定する。
    * Switch 文で SyntheticEventCtor に適切な event オブジェクトを代入する。
2. accumulateSinglePhaseListeners 関数（イベントがバブリングフェーズで捕捉されると仮定して）が実行される。
    1. イベント発生元の Fiber の stateNode(instance).canonical.currentProps から onXXXX（event listener）関数を取り出す。
    2. 親 Tree 上に存在する Fiber を辿り、同様の onXXXX イベントを取り出す。
    3. listeners（{instance, listener, eventTarget}[]）として返す。
3. dispatchQueue（配列）に { event: ReactSyntheticEvent, listeners } オブジェクトを push する。

<details>

<summary>code</summary>

```ts
function extractEvents(
  dispatchQueue: DispatchQueue,
  domEventName: DOMEventName,
  targetInst: null | Fiber,
  nativeEvent: AnyNativeEvent,
  nativeEventTarget: null | EventTarget,
  eventSystemFlags: EventSystemFlags,
  targetContainer: EventTarget,
): void {
  // click -> onClick
  const reactName = topLevelEventsToReactNames.get(domEventName);
  if (reactName === undefined) {
    return;
  }
  let SyntheticEventCtor = SyntheticEvent;
  let reactEventType: string = domEventName;

  switch (domEventName) {
    case 'keypress':
      // firefox にて function キーが反応してしまうので無効にする
      if (getEventCharCode(((nativeEvent: any): KeyboardEvent)) === 0) {
        return;
      }
    /* falls through */
    case 'keydown':
    case 'keyup':
      SyntheticEventCtor = SyntheticKeyboardEvent;
      break;
    case 'focusin':
      reactEventType = 'focus';
      SyntheticEventCtor = SyntheticFocusEvent;
      break;
    case 'focusout':
      reactEventType = 'blur';
      SyntheticEventCtor = SyntheticFocusEvent;
      break;
    case 'beforeblur':
    case 'afterblur':
      SyntheticEventCtor = SyntheticFocusEvent;
      break;
    case 'click':
      // firefox にて右クリックが反応してしまうので無効にする
      if (nativeEvent.button === 2) {
        return;
      }
    // ...
    case 'touchcancel':
    case 'touchend':
    case 'touchmove':
    case 'touchstart':
      SyntheticEventCtor = SyntheticTouchEvent;
      break;
    // ...
    default:
      // Unknown event. This is used by createEventHandle.
      break;
  }

  const inCapturePhase = (eventSystemFlags & IS_CAPTURE_PHASE) !== 0;
  if (
    enableCreateEventHandleAPI &&
    eventSystemFlags & IS_EVENT_HANDLE_NON_MANAGED_NODE
  ) {
    // キャプチャーフェーズで捕捉する event
    const listeners = accumulateEventHandleNonManagedNodeListeners(
      // TODO: this cast may not make sense for events like
      // "focus" where React listens to e.g. "focusin".
      ((reactEventType: any): DOMEventName),
      targetContainer,
      inCapturePhase,
    );
    if (listeners.length > 0) {
      // Intentionally create event lazily.
      const event: ReactSyntheticEvent = new SyntheticEventCtor(
        reactName,         // onClick
        reactEventType,    // click
        null,
        nativeEvent,       // event object
        nativeEventTarget, // event.target
      );
      dispatchQueue.push({event, listeners});
    }
  } else {
    const accumulateTargetOnly =
      !inCapturePhase &&
      (domEventName === 'scroll' || domEventName === 'scrollend');
    // バブリングフェーズで捕捉する event
    const listeners = accumulateSinglePhaseListeners(
      targetInst,
      reactName,
      nativeEvent.type,
      inCapturePhase,
      accumulateTargetOnly,
      nativeEvent,
    );
    if (listeners.length > 0) {
      // Intentionally create event lazily.
      const event: ReactSyntheticEvent = new SyntheticEventCtor(
        reactName,
        reactEventType,
        null,
        nativeEvent,
        nativeEventTarget,
      );
      // { ReactSyntheticEvent, [{instance, listener: (e:ReactSyntheticEvent) => void, lastHostComponent}] }
      dispatchQueue.push({event, listeners});
    }
  }
}
```

</details>

### listenToAllSupportedEvents ~ dispatchEventForPluginEventSystem > dispatchEventsForPlugins > processDispatchQueue

```ts
export function processDispatchQueue(
  dispatchQueue: DispatchQueue,
  eventSystemFlags: EventSystemFlags,
): void {
  const inCapturePhase = (eventSystemFlags & IS_CAPTURE_PHASE) !== 0;
  for (let i = 0; i < dispatchQueue.length; i++) {
    const {event, listeners} = dispatchQueue[i];
    processDispatchQueueItemsInOrder(event, listeners, inCapturePhase);
    //  event system doesn't use pooling.
  }
}
```

### listenToAllSupportedEvents ~ dispatchEventsForPlugins > processDispatchQueue > processDispatchQueueItemsInOrder

```ts
function processDispatchQueueItemsInOrder(
  event: ReactSyntheticEvent,
  dispatchListeners: Array<DispatchListener>,
  inCapturePhase: boolean,
): void {
  let previousInstance;
  if (inCapturePhase) {
    // ...
  } else {
    for (let i = 0; i < dispatchListeners.length; i++) {
      const {instance, currentTarget, listener} = dispatchListeners[i];
      if (instance !== previousInstance && event.isPropagationStopped()) {
        return;
      }
      if (__DEV__ && enableOwnerStacks && instance !== null) {
        // ...
      } else {
        // React Component の listener（onClick）の実行
        executeDispatch(event, listener, currentTarget);
      }
      previousInstance = instance;
    }
  }
}
```
