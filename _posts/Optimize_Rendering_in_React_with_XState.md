
# How to Optimize Rendering in React with XState Machines

When using React and XState machines, updates to the machine’s context can lead to a re-render of the entire subtree. Here’s how to optimize rendering and avoid unnecessary re-renders.

---

## Key Strategies to Optimize Rendering

### 1. Use Memoization for Static Parts
Use `React.memo` to prevent unnecessary re-renders of components or subcomponents in your rendering tree.

```tsx
import React from "react";

const RenderEngine = React.memo(({ jsonNode }) => {
  // Your existing RenderEngine logic
});
```

This ensures `RenderEngine` only re-renders when its `jsonNode` prop changes by reference.

---

### 2. Use `useSelector` for Fine-Grained State Updates
Leverage `useSelector` from `@xstate/react` to extract only the specific part of the machine's context your component depends on.

Example:

```tsx
const jsonNode = useSelector(actorRef, (snapshot) => snapshot.context.nodes);
```

This ensures only updates to `nodes` in the context trigger a re-render.

---

### 3. Avoid Updating the Entire JSON Node
When updating the JSON node in your XState machine, modify only the specific part of the JSON that has changed to avoid unnecessary reference changes.

Example using `immer`:

```ts
assign({
  nodes: (context, event) =>
    produce(context.nodes, (draft) => {
      const targetNode = draft.find((node) => node.id === event.id);
      if (targetNode) {
        targetNode.props[event.propKey] = event.value;
      }
    }),
});
```

---

### 4. Use Keys for Efficient Reconciliation
Ensure every element in your JSON has a unique `id`, which is passed as the `key` prop in React. React uses the `key` to optimize DOM reconciliation.

Example:

```tsx
<RenderEngine key={jsonNode.id} jsonNode={jsonNode} />
```

---

### Example Integration

Here’s how you can integrate these optimizations in your `MyDataScreen` component:

```tsx
import React from "react";
import tw from "twrnc";
import { View, Text } from "react-native";
import SafeColorBackground from "../../components/SafeColorBackground";
import RenderEngine from "../../renderEngine/RenderEngine";
import { useSelector, useActorRef } from "@xstate/react";
import renderEngine from "../../xstate/machines/renderEngine";

const MyDataScreen = () => {
  const actorRef = useActorRef(renderEngine, { input: { entity: "caca" } });
  const jsonNode = useSelector(actorRef, (snapshot) => snapshot.context.nodes);

  console.log("jsonNode", jsonNode);

  return (
    <SafeColorBackground backgroundColor="white">
      {jsonNode ? <RenderEngine jsonNode={jsonNode} /> : <Text>Loading...</Text>}
    </SafeColorBackground>
  );
};

export default React.memo(MyDataScreen);
```

---

### Summary

1. **JSON Changes Lead to Re-renders**:
   - By default, any change in the `jsonNode` object triggers a re-render of the `RenderEngine` subtree.

2. **Optimizations**:
   - Use `React.memo` and fine-grained state selection (`useSelector`) to minimize unnecessary renders.
   - Update only specific parts of the JSON node in your XState machine to reduce reference changes.
   - Use unique `key` props for efficient React reconciliation.

By applying these strategies, your application will render efficiently even when dynamically updating XState machine contexts. 
