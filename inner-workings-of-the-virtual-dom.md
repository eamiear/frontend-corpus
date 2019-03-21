Virtual DOM \(VDOM aka VNode\) is magical ✨ but is also complex and hard to understand😱.[React](https://facebook.github.io/react/),[Preact](https://preactjs.com/)and similar JS libraries use them in their core. Unfortunately I couldn’t find any good article or doc that explains it in a detailed-yet-simple-to-understand fashion. So I thought of writing one myself.

> Note: This is a LONG post. I’ve added tons of pictures to make it simple but it also makes the post appear even longer.

> I’m using
> [Preact’s](https://github.com/developit/preact/)
> code and VDOM as it is small and you can look at it yourself with ease in the future.
> **But I think most of the concepts applies to React as well.**

> **My hope is that once you read this, you’ll be able to better understand and hopefully contribute to libraries like React and Preact.**

In this blog, I’ll take a simple example and go over various scenarios to give you an idea as to how they actually work. Specifically, I’ll go over:

1. Babel and JSX
2. Creating VNode — A single virtual DOM element
3. Dealing with components and sub-components
4. Initial rendering and creating a DOM element
5. Re-rendering
6. Removing DOM element.
7. Replacing a DOM element.

### The app: {#6fa5}

The app a simple[filterable Search app](http://codepen.io/rajaraodv/pen/BQxmjj)that contains two components “**FilteredList**” and “**List**”. The List renders a list of items \(default: “California” and “New York”\). The app has a search field that filters the list based on the characters in the field. Pretty straight forward.

![](https://cdn-images-1.medium.com/max/1000/1*VKN_mqVAzDfqcTU1PM47wg.png)

Picture of the app \(Click to zoom and see the details\)

> Live app:
> [http://codepen.io/rajaraodv/pen/BQxmjj](http://codepen.io/rajaraodv/pen/BQxmjj)

### The Big Picture {#a446}

At a high-level, we write components in JSX\(html in JS\), that gets converted to pure JS by CLI tool[Babel](http://babeljs.io/). Then Preact’s “h” \([hyperscript](https://github.com/dominictarr/hyperscript)\) function, converts it into VDOM tree \(aka VNode\). And finally Preact’s Virtual DOM algorithm, creates real DOM from the VDOM that creates our app.

![](https://cdn-images-1.medium.com/max/1000/1*imVzO1P2k1cIuf04_rGYGw.png)

The big picture

**Before we get into the weeds of the VDOM lifecycle, let’s understand JSX as it provides the starting point for the library.**

### 1. Babel And JSX {#eed0}

In React, Preact like libraries, there is no HTML and instead**everything is JavaScript**. So we need to write even the HTML in JavaScript. But writing DOM in pure JS is a nightmare!😱

For our app we’ll have to write HTML like below:

> Note: I’ll explain “h” soon

![](https://cdn-images-1.medium.com/max/1000/1*KExHlJcp3om5h1JFA09Ahw.png)

![](https://cdn-images-1.medium.com/max/1000/1*uOSYO2xCEcuGWNzKNjKUnA.png)

That’s where JSX comes in. JSX essentially allows us to write HTML in JavaScript! And also allows us to use JS within that by curly braces{}.

JSX helps us easily write our components like below:

![](https://cdn-images-1.medium.com/max/1000/1*zUYkQpIdUdPp_SIOnEX8Bg.png)

![](https://cdn-images-1.medium.com/max/1000/1*Xc92YmYZeaJ6oX0PCGYfyw.png)

#### Converting JSX tree to JavaScript {#602e}

JSX is cool but it’s not a valid JS, but ultimately we need REAL DOM. JSX only helps in writing a_representation_of real DOM and otherwise it’s useless.

So we a way need to convert into a corresponding JSON object \(VDOM, which is also a tree\) so we can eventually use it as an input to create real DOM. We need a function to do that.

And that function is the[“h” function](https://github.com/developit/preact/blob/master/src/h.js)in Preact. It’s the equivalent to “[React.createElement](https://facebook.github.io/react/docs/react-api.html#createelement)” in React.

> “h” stands for
> [hyperscript](https://github.com/dominictarr/hyperscript)
>  — one of the first libs to create HTML in JS \(VDOM\)

But how to convert JSX into “h” function calls? And that’s where[Babel](http://babeljs.io/)comes in. Babel simply goes through each JSX node and converts them to “h” function calls.

![](https://cdn-images-1.medium.com/max/1000/1*TC2uwjc4mBrVHzOd0AeShw.png)

#### **Babel JSX \(React Vs Preact\)** {#c070}

By default, Babel converts JSX to React.createElement calls because it defaults to React.

![](https://cdn-images-1.medium.com/max/1000/1*ylur5c7_DCWyJ_RHS_ndYg.png)

![](https://cdn-images-1.medium.com/max/2000/1*ylur5c7_DCWyJ_RHS_ndYg.png)

LEFT: JSX RIGHT: React version of JS \(click to Zoom\)

But we can easily change the name of the function to anything we want \(like “h” for Preact\) by adding “[Babel Pragma](https://babeljs.io/docs/plugins/transform-react-jsx/)” like below:

```
Option 1:


//.babelrc


{   "plugins": [


      ["transform-react-jsx", { "pragma": "h" }]


     ] 


}
```

```
Option 2:


//Add the below comment as the 1st line in every JSX file


/** @jsx h */
```

![](https://cdn-images-1.medium.com/max/1500/1*LmJWuVB_5OIXOW99b5_Yfg.png)

“h” —With Babel Pragma \(Click to Zoom\)

#### Main Mount To real DOM {#f240}

Not only the code in “render” methods of the components are converted to “h” functions, but also the starting mount!

**And this is where the execution start and everything begins!**

```
//Mount to real DOM


render(
<
FilteredList/
>
, document.getElementById(‘app’));
```

```
//Converted to "h":


render(
h(FilteredList)
, document.getElementById(‘app’));
```

#### The Output of “h” function {#5d8d}

The “h” function takes the output of JSX and creates something called a “VNode”\(React’s “createElement” creates ReactElement\). A Preact’s “VNode” \(or a React’s “Element”\) is simply a JS object representation of a single DOM node with it’s properties and children.

It looks like this:

```
{


   "nodeName": "",


   "attributes": {},


   "children": []


}
```

For example, VNode for our app’s Input looks like this:

```
{


   "nodeName": "input",


   "attributes": {


    "type": "text",


    "placeholder": "Search",


    "onChange": ""


   },


   "children": []


  }
```

> **Note: “h” function doesn’t create the entire tree!**
> It simply creates JS object for a given node. But since the “
> **render**
> ” method already has the DOM JSX in a tree fashion, the end result will be a VNode with children and grand children that looks like a tree.

> _**Reference Code:**_

> _**“h” :**_
> [_https://github.com/developit/preact/blob/master/src/h.js_](https://github.com/developit/preact/blob/master/src/h.js)

> **VNode**
> :
> [https://github.com/developit/preact/blob/master/src/vnode.js](https://github.com/developit/preact/blob/master/src/vnode.js)

> _**“render”:**_
> [_https://github.com/developit/preact/blob/master/src/render.js_](https://github.com/developit/preact/blob/master/src/render.js)

> **“buildComponentFromVNode:**
> [https://github.com/developit/preact/blob/master/src/vdom/diff.js\#L102](https://github.com/developit/preact/blob/master/src/vdom/diff.js#L102)

OK, let’s see how Virtual DOM works.

### Virtual DOM Algorithm Flowchart For Preact {#0480}

In the flowchart below shows how components \(and child components\) are created, updated and deleted by Preact. It also shows when lifecycle events like “componentWillMount” and so on are called.

> Note: We’ll go over each section in a step-by-step manner so don’t worry if it looks complicated

![](https://cdn-images-1.medium.com/max/1500/1*TF0TZszVwpYc1Pba7Dbk7Q.png)

Yes, it’s hard to understand all at once. So let’s go over various sections of the flowchart by going through various scenarios in a step-by-step manner.

> Note: I’ll highlight sections of the lifecycle in “yellow” when discussing specific steps.

### Scenario 1: Initial Creation Of The App {#e272}

#### 1.1 — Creating VNode \(Virtual DOM\) For A Given Component {#7d34}

The highlighted section shows the initial loop that creates VNode \(Virtual DOM\) tree for a given component. Note that this doesn’t create VNode for sub-components \(that’s a different loop\).

![](https://cdn-images-1.medium.com/max/1500/1*JVwKJe82S1wHFqzr17DQ4g.png)

Yellow highlight shows the VNode creation

The picture below shows what happens when our app loads for the first time. The library ends up creating a VNode with children and attributes for the main FilteredList component.

> Note: Along the way it also calls “componentWillMount” and “render” lifecycle methods \(see the green blocks in the picture above\).

![](https://cdn-images-1.medium.com/max/1000/1*CI75nMf-7j4ldxPJQ8nciQ.png)

![](https://cdn-images-1.medium.com/max/2000/1*CI75nMf-7j4ldxPJQ8nciQ.png)

\(click to zoom\)

At this point, we have a VNode that has a “**div**” parentNode that has an “**input**” and a “**List**” child nodes.

> **Reference code:**

> _Most lifecycle events like: componentWillMount, render and so on:_
> [_https://github.com/developit/preact/blob/master/src/vdom/component.j_](https://github.com/developit/preact/blob/master/src/vdom/component.js#L101)
> _s_

#### 1.2 — If Not A Component, create a REAL DOM {#1de0}

In this step, it’ll simply create real DOM for the parent node \(div\) and repeat process for child nodes \(“input” and “List”\).

![](https://cdn-images-1.medium.com/max/1500/1*X1tcv3n47OuAExoTBu3Q5A.png)

Highlighted loop shows REAL DOM creation for child components

At this point, we have just “div” as shown in the picture below:

![](https://cdn-images-1.medium.com/max/1000/1*l-FmgMheFRYFuhCJQA4gcQ.png)

> **Reference code:**

> document.createElement:
> [https://github.com/developit/preact/blob/master/src/dom/recycler.js](https://github.com/developit/preact/blob/master/src/dom/recycler.js)

#### 1.3 — Repeat for all children {#41d3}

At this point, the loop is repeated for all children. In our app, it’ll be repeated for “input” and “List” items.

![](https://cdn-images-1.medium.com/max/1500/1*XadPNReNXRQyIl6FJOpPlQ.png)

Loop for each children

#### 1.4 — Process Child And Append To Parent. {#0324}

In this step, we’ll process leaf. Since “input” has a parent \(“div”\), we’ll append input as a child to div. Then the control stops and return to create “List” \(which is the 2nd child of “div”\).

![](https://cdn-images-1.medium.com/max/1500/1*iLQQTuRdKIhlsh5Vgaox0w.png)

Finish processing leaf

At this point, our app looks like below:

![](https://cdn-images-1.medium.com/max/1000/1*uyA28eM-UBnU0SSX8vPSdg.png)

> Note: that after “input” is created, since it doesn’t have any children, it doesn’t immediately loop and create “List”! Instead it’ll first append “input” to the parent “div” and then goes back to process “List”

> **Reference code:**

> appendChild:
> [https://github.com/developit/preact/blob/master/src/vdom/diff.js](https://github.com/developit/preact/blob/master/src/vdom/diff.js)

#### 1.5 Process child component\(s\) {#93ed}

The control goes back to step 1.1 and starts all 0ver again for “List” component. But since “List” is a component, it calls the**render**method of the “List” component to get new set of VNodes that look like below.

![](https://cdn-images-1.medium.com/max/1500/1*XadPNReNXRQyIl6FJOpPlQ.png)

Repeat everything for a child “component”

That loop completes for the List component and returns List’s VNode that looks like below:

![](https://cdn-images-1.medium.com/max/1000/1*PbgdX1qNPoHh5Ok3yY83gQ.png)

> **Reference Code:**

> **“buildComponentFromVNode:**
> [https://github.com/developit/preact/blob/master/src/vdom/diff.js\#L102](https://github.com/developit/preact/blob/master/src/vdom/diff.js#L102)

#### 1.6 Repeat steps 1.1 through 1.4 for all the Child Nodes {#a54f}

It’ll repeat the above steps again for each node. Once it reaches the leaf node, it appends it to the node’s parent and repeats the process.

![](https://cdn-images-1.medium.com/max/1500/1*Pwkyw7P2bbpEL9zQB1YaMA.png)

Keep doing the same until all child and parent are created and appended

The below picture shows how each node is added \(hint: depth-first\).

![](https://cdn-images-1.medium.com/max/2600/1*e6rihM5IE3PRedd311iinA.png)

How REAL DOM tree is created by the VDOM algorithm

#### 1.7 Finish processing {#4412}

At this point, It’s done processing. It simply calls “componentDidMount” for all the components \(starting from child components to parent components\) and stops.

![](https://cdn-images-1.medium.com/max/1500/1*xRspRlx0WM-y2Au0zyegog.png)

> **Important Note:**
> Once everything is done, a reference to the real DOM is added to each of the component instances. This reference is used for remaining updates \(create, update, delete\) to compare and avoid recreating the same DOM nodes.

### SCENARIO 2: Delete Leaf Node {#73fa}

Say we typed “cal” and hit enter. This will remove the 2nd list node, a leaf node \(New York\) while keeping all other parent nodes.

![](https://cdn-images-1.medium.com/max/1000/1*AyQmTTZFnfC_VKv23D8FrA.png)

![](https://cdn-images-1.medium.com/max/1250/1*AyQmTTZFnfC_VKv23D8FrA.png)

Let’s see how the flow looks for this scenario.

#### 2.1 Create VNodes like before. {#640e}

After initial rendering, every change in the future is an “update”. When it comes to creating VNodes, the update cycle works very similar to create cycle and**creates VNodes all over again.**

But since it’s an update \(and not creation\) of the component, it makes “componentWillReceiveProps”, “shouldComponentUpdate”, and “componentWillUpdate” calls to each component and sub-component.

**In addition, update cycle, doesn’t recreate DOM elements if those elements are already there.**

![](https://cdn-images-1.medium.com/max/1500/1*pCYNGbZL_zT1gbMtXYVt9A.png)

Updating component lifecycle

> **Reference Code**

> **removeNode:**
> [https://github.com/developit/preact/blob/master/src/dom/index.js\#L9](https://github.com/developit/preact/blob/master/src/dom/index.js#L9)

> **insertBefore:**
> [https://github.com/developit/preact/blob/master/src/vdom/diff.js\#L253](https://github.com/developit/preact/blob/master/src/vdom/diff.js#L253)

#### 2.2 Use the reference real DOM node and avoid creating duplicate nodes {#d306}

As mentioned earlier, each component has a reference to corresponding real DOM tree that was created during initial loading. The picture below shows how references look for our app at this point.

![](https://cdn-images-1.medium.com/max/1500/1*JjV4E5Ag9ogDvH9ILWDO2Q.png)

Showing references b/w previous DOM and each component

And when VNodes created, each VNode’s attributes are compared w/ the attributes of the REAL DOM at that node**. If real DOM exists, the loop moves on to the next node.**

![](https://cdn-images-1.medium.com/max/1500/1*fHYrhlaGOJKnWTzn0YrQ8g.png)

Real DOM “already exists” lifecycle \(during update\)

> **Reference Code**

> innerDiffNode:
> [https://github.com/developit/preact/blob/master/src/vdom/diff.js\#L185](https://github.com/developit/preact/blob/master/src/vdom/diff.js#L185)

#### 2.3 Remove node if there are extra nodes in the REAL DOM {#e802}

The picture below shows the difference in REAL DOM V/s VNode.

![](https://cdn-images-1.medium.com/max/1000/1*ALjitFLcW_lIHvaswCp0GA.png)

\(click to zoom\)

And since there is a difference, the “New York” node in REAL DOM is removed by the algorithm as shown in the workflow below. The algorithm also calls “componentDidUpdate” lifecycle event once everything is done.

![](https://cdn-images-1.medium.com/max/1500/1*m74VaQdTuuonA_R-LkT3xw.png)

Remove DOM node lifecycle

### SCENARIO 3 — Unmounting Entire Component {#efef}

Use case: Let’s say if we typed**blabla**in the filter, since it doesn’t match “California” or “New York”, we won’t render the child component “List” at all. This means, we need to unmount the entire component.

![](https://cdn-images-1.medium.com/max/1000/1*Ki5tKGKiRI4P_ma_fz3chQ.png)

List component is not removed if there are no results

![](https://cdn-images-1.medium.com/max/1000/1*5GheZk3dZ7st_mvBcWG4nw.png)

“render” method of FilteredList

Deleting a component is similar to Deleting a single node. Except, when we delete a node that has a reference to a component, then the framework calls “componentWillUnmount” and then recursively deletes all the DOM elements. After all the elements are removed from the real DOM, it calls “componentDidUnmount” method of the referenced component.

The picture below shows the reference to “List” component on the real DOM “ul”.

![](https://cdn-images-1.medium.com/max/1000/1*yZ3TSnTRf95mGSoR-HiwHw.png)

sadf

The below picture highlights the section in the flowchart to show how deleting/unmounting a component works.

![](https://cdn-images-1.medium.com/max/1500/1*QGGL_W7zEr3FtR1VODXt_w.png)

Remove and unmount component

> **Reference code**

> unmountComponent:
> [https://github.com/developit/preact/blob/master/src/vdom/component.js\#L250](https://github.com/developit/preact/blob/master/src/vdom/component.js#L250)

#### **Final Notes:** {#2ab6}

I hope that this post gave you enough idea as to how Virtual DOM works \(at least in Preact\).

Please note that while these scenarios covers major ones, I haven’t covered some of the optimizations in the code.

🙏🏼 Thank you!

#### If this was useful, please click the clap 👏 button down below a few times to show your support! ⬇⬇⬇ 🙏🏼 {#e8b6}



