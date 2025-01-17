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
3. listenToAllSupportedEvents の実行により rootContainerElement の eventListener へ全ての event（"click", "touch", etc...）を登録する。

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

### createFiberRoot

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

### listenToNativeEvent

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

### addTrappedEventListener

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

### createEventListenerWrapperWithPriority

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

### dispatchEvent
