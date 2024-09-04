# HODL Project Documentation - Django Backend

## Tree Structure Functionality

The HODL project implements a hierarchical tree structure for managing user relationships, likely for a referral or multi-level marketing system. This functionality is implemented in the `tree.py` file within the `user` app.

### Overview

The tree structure is implemented using the `django-tree-queries` package, which provides efficient querying of hierarchical data in Django models. The `CustomerUserProfile` model (not shown in the provided code) is assumed to be using this package.

### Key Functions

#### 1. get_parent(node)

This function retrieves the immediate parent of a given node in the tree.

```python
def get_parent(node):
    ancestors = node.ancestors()
    parent = ancestors.last() if ancestors else None
    return parent
```

- **Input**: A node (instance of `CustomerUserProfile`)
- **Output**: The immediate parent node, or `None` if the node has no parent
- **Usage**: Use this function to find the direct upline of a user in the hierarchy

#### 2. get_direct_childs(node)

This function retrieves all direct children (one level down) of a given node.

```python
def get_direct_childs(node):
    all_descendants = list(node.descendants())
    direct_childs = [
        descendant
        for descendant in all_descendants
        if descendant.tree_depth == node.tree_depth + 1
    ]
    return direct_childs
```

- **Input**: A node (instance of `CustomerUserProfile`)
- **Output**: A list of direct child nodes
- **Usage**: Use this function to find all direct downlines of a user

#### 3. find_ancestors(node, lvl=None)

This function retrieves ancestors (upline) of a given node, optionally limited to a specified number of levels.

```python
def find_ancestors(node, lvl=None):
    ancestors = list(node.ancestors())
    if lvl is None:
        return ancestors
    else:
        return ancestors[-lvl:] if len(ancestors) >= lvl else ancestors
```

- **Input**: 
  - `node`: An instance of `CustomerUserProfile`
  - `lvl` (optional): Number of levels to go up in the hierarchy
- **Output**: A list of ancestor nodes
- **Usage**: Use this function to find the upline of a user, optionally limited to a specific number of levels

#### 4. find_descendants(node, lvl=None)

This function retrieves descendants (downline) of a given node, optionally limited to a specified number of levels.

```python
def find_descendants(node, lvl=None):
    all_descendants = node.descendants()
    if lvl is None:
        return list(all_descendants)
    else:
        up_to_lvl_levels_down = [
            descendant
            for descendant in all_descendants
            if 0 < descendant.tree_depth - node.tree_depth <= lvl
        ]
        return up_to_lvl_levels_down
```

- **Input**:
  - `node`: An instance of `CustomerUserProfile`
  - `lvl` (optional): Number of levels to go down in the hierarchy
- **Output**: A list of descendant nodes
- **Usage**: Use this function to find the downline of a user, optionally limited to a specific number of levels

### Usage Examples

```python
# Assuming 'user' is an instance of CustomerUserProfile

# Get the user's direct upline
parent = get_parent(user)

# Get all direct downlines of the user
direct_downlines = get_direct_childs(user)

# Get the user's upline, up to 3 levels
upline = find_ancestors(user, lvl=3)

# Get the user's entire downline, up to 5 levels deep
downline = find_descendants(user, lvl=5)
```

### Notes

- The `CustomerUserProfile` model should be set up with the `django-tree-queries` package for these functions to work correctly.
- The tree structure assumes a single-parent hierarchy, where each node has at most one parent.
- These functions provide efficient querying of the tree structure, which is crucial for performance in large hierarchical datasets.

For more information on setting up and using the `django-tree-queries` package, refer to its official documentation.