My tree is made out of 3 internal components:

* **FSRoot** *(see* [*FSRoot.js*](https://github.com/DAB0mB/react-fs-tree/blob/fc0a0b3b2042e312d4238e0660c161f7744dd732/src/index.js)*)*— This is where the tree starts to grow from. It’s a container that encapsulates internal props which are redundant to the user (like props.rootNode, props.parentNode, etc) and exposes only the relevant parts (like props.childNodes, props.onSelect, etc). It also contains a <style/\> tag which rules that are relevant nested components.
* **FSBranch** *(see* [*FSBranch.js*](https://github.com/DAB0mB/react-fs-tree/blob/fc0a0b3b2042e312d4238e0660c161f7744dd732/src/FSBranch.js)*)* — A branch contains the list that will iterate through the nodes. The branch is what will give the tree the staircase effect and will get further away from the edge as we go deeper. Any time we reveal the contents of a node with child nodes, a new nested branch should be created.
* **FSNode** *(see* [*FSNode.js*](https://github.com/DAB0mB/react-fs-tree/blob/fc0a0b3b2042e312d4238e0660c161f7744dd732/src/FSNode.js)*)* — The node itself. It will present the given node’s metadata: its name, its mode (added, deleted or modified), and its children. This node is also used as a controller to directly control the node’s metadata and update the view right after. More information about that further this article.

A relational diagram of the tree

The recursion pattern in the diagram above is very clear to see. Programmatically speaking, this causes a problematic situation where each module is dependent on one another. So before FSNode.js was even loaded, we import it in FSBranch.js which will result in an undefined module.

Recursive import — the wrong way

There are two ways to solve this problem:

* Switching to CommonJS and move the require() to the bottom of the first dependent module — which I’m not gonna get into. It doesn’t look elegant and it doesn’t work with some versions of Webpack; during the bundling process all the require() declarations might automatically move to the top of the module which will force\-cause the issue again.
* Having a third module which will export the dependent modules and will be used at the next event loop — some might find this an anti pattern but I like it because we don’t have to switch to CommonJS and it’s highly compatible with Webpack’s strategy.

The following code snippet demonstrates the second preferred way of solving recursive dependency conflict:

Recursive import — the right way

There are two methods to implement the staircase effect:

* Using a floating tree — where each branch has a constant left\-margin and completely floats.
* Using a padded tree — where each branch doesn’t move further away but has an incremental padding.

A floating tree makes complete sense. It nicely vertically aligns the nodes within it based on the deepness level we’re currently at. The deeper we go the further away we’ll get from the left edge, which will result in this nice staircase effect.

A floating tree

However, as you can see in the illustrated tree, when selecting a node it will not be fully stretched to the left, as it completely floats with the branch. The solutions for that would be a padded tree.

Unlike the floating tree, each branch in the padded tree would fully stretch to the left, and the deeper we go the more we gonna increase the pad between the current branch and the left edge. This way the nodes will still be vertically aligned like a staircase, but now when we select them, the highlight would appear all across the container. It’s less intuitive and slightly harder to implement, but it does the job.

A padded tree

Programmatically speaking, this would require us to pass a counter that will indicate how deep the current branch is (n), and multiply it by a constant value for each of its nodes (x) (See [implementation](https://github.com/DAB0mB/react-fs-tree/blob/475c394c51dca52dda205dd1ceaba2ede679609b/src/fs-node.js#L263)).

One of the things that I was looking to have in my tree was an easy way to update it, for example, if one node was selected, deselected the previous one, so selection can be unique. There are many ways that this could be achieved, the most naive one would be updating one of the node’s data and then resetting the state of the tree from its root.

There’s nothing necessarily bad with that solution and it’s actually a great pattern, however, if not implemented or used correctly, this can cause the entire DOM tree to be re\-rendered, which is completely unnecessary. Instead, why not just use the node’s component as a controller?

You heard me right. Directly grabbing the reference from the React.Component’s callback and use the methods on its prototype. Sounds tricky, but it works fast and efficiently (see [implementation](https://github.com/DAB0mB/react-fs-tree/blob/475c394c51dca52dda205dd1ceaba2ede679609b/src/fs-node.js#L146)).

An example for unique selection

One thing to note is that since the controllers are hard\-wired to the view, hypothetically speaking we wouldn’t be able to have any controllers for child nodes of a node that is not revealed (`node.opened === false`). I’ve managed to bypass this issue by using the React.Component’s constructor directly. This is perfectly legal and no error is thrown, unless used irresponsibly to render something, which completely doesn’t make sense (`new FSNode(props)`; see [implementation](https://github.com/DAB0mB/react-fs-tree/blob/475c394c51dca52dda205dd1ceaba2ede679609b/src/fs-node.js#L330)).

A program can be written in many ways. I know that my way of implementing a tree view can be very distinct, but since all trees should be based around recursion, you can take a lot from what I’ve learnt.

Below is the final result of the tree that I’ve created. Feel free to visit its [Github page](https://github.com/DAB0mB/react-fs-tree) or grab a copy using NPM.

$ npm install react\-fs\-tree