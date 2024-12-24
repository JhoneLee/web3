# react 分平台打点设计

这篇设计方案详细讲述了如何在 React 平台中实现打点（分析事件）的功能，并且给出了如何封装、替换事件、分平台的具体实现方案。下面是对这个设计方案的解析，补充了一些具体实现细节和注意事项。

### 1. **封装 Trace 组件**

**Trace 组件** 的核心职责是将用户的行为和 UI 元素的状态变化关联起来，并且触发打点事件。它通过 Context 和事件绑定的方式，将打点功能注入到子组件中。

```javascript
// Trace 组件通过 Context 和 Provider 注入打点逻辑
export const TraceContext = createContext < ITraceContext > {};

function _Trace({
  children,
  logImpression,
  eventOnTrigger,
  logPress,
  logFocus,
  logKeyPress,
  directFromPage,
  screen,
  page,
  section,
  element,
  modal,
  properties,
}: PropsWithChildren<TraceProps & ITraceContext>): JSX.Element {
  const id = useId();
  const { useIsPartOfNavigationTree, shouldLogScreen } =
    useAnalyticsNavigationContext();
  const isPartOfNavigationTree = useIsPartOfNavigationTree();
  const parentTrace = useTrace();

  const events = useMemo(
    () => getEventsFromProps(logPress, logFocus, logKeyPress),
    [logFocus, logKeyPress, logPress]
  );

  const modifiedChildren =
    events.length > 0 ? (
      <TraceContext.Consumer>
        {(consumedProps): ReactNode =>
          React.Children.map(children, (child) => {
            if (!React.isValidElement(child)) {
              return child;
            }

            return React.cloneElement(
              child,
              getEventHandlers(
                child,
                consumedProps,
                events,
                eventOnTrigger ?? SharedEventName.ELEMENT_CLICKED,
                element,
                properties
              )
            );
          })
        }
      </TraceContext.Consumer>
    ) : (
      children
    );

  return (
    <TraceContext.Provider value={combinedProps}>
      {modifiedChildren}
    </TraceContext.Provider>
  );
}
```

**关键点**：

- **Context**：通过`TraceContext`和`Provider`将打点数据传递给子组件。
- **事件监听**：通过`getEventHandlers`给每个直接子组件添加打点事件处理逻辑。
- **事件处理**：当用户触发事件时，执行原有的事件处理逻辑，并且触发打点事件。

### 2. **函数替换功能事件（getEventHandlers）**

**getEventHandlers** 函数的目的是替换原有的事件处理程序，加入打点功能，同时保留原事件的逻辑。这个函数不仅执行事件处理，还会将打点数据发送出去。

```javascript
export function getEventHandlers(
  child: React.ReactElement,
  consumedProps: ITraceContext,
  triggers: string[],
  eventName: string,
  element?: string,
  properties?: Record<string, unknown>
): Partial<Record<string, (e: Event) => void>> {
  const eventHandlers: Partial<Record<string, (e: Event) => void>> = {};

  for (const event of triggers) {
    eventHandlers[event] = (eventHandlerArgs: unknown): void => {
      // 执行宿主组件的原始事件逻辑
      child.props[event]?.apply(child, [eventHandlerArgs]);

      // 发送打点数据
      analytics.sendEvent(eventName, {
        element,
        ...consumedProps,
        ...properties,
      });
    };
  }

  return eventHandlers;
}
```

**关键点**：

- **合并原事件与打点事件**：新的事件处理函数执行原有事件的处理逻辑并发送打点数据。
- **触发打点数据的发送**：通过`analytics.sendEvent`发送打点数据。

### 3. **打点事件的发送**

根据不同平台（Web 和 Native）的需求，打点事件的发送逻辑需要进行平台区分。通过 Webpack 的配置，可以为不同平台提供不同的实现文件。

#### Webpack 配置

通过 Webpack 配置 `resolve.extensions`，根据后缀名区分平台：

```javascript
// webpack.config.ts
webpackConfig.resolve.extensions.unshift(".web.tsx");
webpackConfig.resolve.extensions.unshift(".web.ts");
webpackConfig.resolve.extensions.unshift(".web.js");
```

#### 平台区分代码

为不同平台编写专属的打点代码文件：

```javascript
// analytics.native.ts
export const analytics = {
  sendEvent: (eventName, data) => {
    // Native平台的打点发送逻辑
  },
};

// analytics.web.ts
export const analytics = {
  sendEvent: (eventName, data) => {
    // Web平台的打点发送逻辑
  },
};

// analytics.ts
export const analytics = {
  sendEvent: (eventName, data) => {
    // 打桩函数，防止单元测试触发打点
  },
};
```

#### 使用代码

在业务代码中直接导入`analytics`，根据不同的环境，Webpack 会加载不同的实现：

```javascript
import { analytics } from "utilities/src/telemetry/analytics/analytics";

// 使用打点数据
analytics.sendEvent("PAGE_VIEW", { page: "home" });
```

#### 单元测试

在单元测试中，`analytics.ts` 会作为打桩函数，不会发送真实的打点数据。

### 总结

该设计的核心是将打点逻辑从 UI 组件中解耦出来，采用 Context 和事件替换的方式注入到子组件中。通过 Webpack 配置实现不同平台的打点逻辑区分，确保代码可以根据不同平台加载相应的文件，避免在单元测试中触发真实的打点事件。

这种方案具有很好的灵活性，支持 Web 和 Native 平台分开打点，同时也保留了原有的 UI 事件逻辑，确保在执行打点时不干扰用户体验。
