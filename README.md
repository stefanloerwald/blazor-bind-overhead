# The issue

There is a significant overhead when using events or `@bind` in Blazor, as discussed in [aspnetcore#18919](https://github.com/dotnet/aspnetcore/issues/18919)

Consider this code

`Parent.razor`
```html
<Child @bind-Value="Value" />
@code {
   private int Value { get; set; }
}
```

`Child.razor`
```html
<button @onclick="() => Value += 1; ValueChanged.InvokeAsync(Value);">Click me</button>
@code {
   [Parameter] public int Value {get;set;}
   [Parameter] public EventCallback<int> ValueChanged {get;set;}
}
```

In this example, clicking the button forces a render of the `Parent` component.
This is unnecessary, as the only place where `Parent.Value` is used is inside the `Child` component, which already has the correct value.
We could therefore avoid the render of `Parent`.
The render does not change anything in the DOM, but still evaluates the entire `Parent` component, and all its children.
When the number of children grows, this overhead eventually becomes noticable.

# Demo

To demonstrate this overhead, there are four components:

- `BindTo.razor`: The component that contains a button to change the bound value.
- `Reference.razor`: A simple component with just one child. No significant render delay here, as only one parent and one child component are re-rendered unnecessarily.
- `ShowsRenderOverhead.razor`: A component with a lot of child components. There is a significant render delay here, even though only one child actually produces output to the DOM.
- `ShowsNorenderWorkaround.razor`: A component to demonstrate that the culprit is indeed the implicit render: By preventing any reference to `this` in the `EventCallback` passed to the child components, we prevent the render cycle.
