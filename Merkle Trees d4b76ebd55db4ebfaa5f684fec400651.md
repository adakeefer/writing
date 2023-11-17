# Merkle Trees

Created: October 21, 2023 10:11 AM
Tags: CS

Merkle trees are a special form of binary tree in which each node contains a hash. This hash is special because it describes the content of that node **************and all child nodes below it**************. The Merkle root (root node of a Merkle tree) can be used to verify, in constant time, if two trees are equivalent (possessing the same values and structure). 

Think of it as a digital fingerprint for a tree. Each is unique, and if two fingerprints are identical they must be the same fingerprint and therefore the same tree. This becomes extremely useful when you start dealing with large trees that require repeated verification. For instance, when synchronizing database replicas or verifying cryptocurrency transactions in a peer-to-peer (P2P) network.

Let’s take a look under the hood.

# Merkle Tree Structure

To create a merkle tree each node must have the following information:

1. Hash of it’s own data
2. Hash of the left subtree’s data
3. Hash of the right subtree’s data

Knowing this, a node can concatenate all of these values and hash them again to produce a new hash. This hash will describe all of the data in that node and all of its subtrees.

In order to get a node’s hash we first need the hash of each subtree’s data. How can we get this? By the definition above, if our subtree is a merkle tree all we need to do is look at the root of the subtree, aka a direct child of our parent node. So we first need to compute the merkle tree for all our subtrees, then concatenate those values with our root value and hash them all together.

We can achieve this via Postorder Traversal, where each node is visited (hashed) after all of the nodes in the left subtree are visited and all the nodes in the right subtree are visited. Say we have a function, *_hash,* which accepts a value v and spits out a hash h. Then computing the merkle tree looks something like this:

```python
def merkleHash(node):
            if not node:
                return '#'
            merkleLeft = merkleHash(node.left)
            merkleRight = merkleHash(node.right)
            toHash = merkleLeft + str(node.val) + merkleRight
            node.merkle = _hash(toHash)
            return node.merkle
```

Notice we supply a dummy value ‘#’ if we do not have a complete binary tree and a node may not have a left, right, or any subchildren. We also assume a binary tree here, but this could work for an n-ary tree so long as we visit each subtree consistently left to right. Finally we are setting a hash value on each node as we visit it here. This works, but it conceivably doubles the space our tree takes up (or more, depending on the original datatype). If we just want the merkle root and don’t need to compare subtrees, we only need to remember the hashes of our subtrees and can forget any of the previously computed hashes.

```python
def merkleHash(node):
            if not node:
                return '#'
            merkleLeft = merkleHash(node.left)
            merkleRight = merkleHash(node.right)
            toHash = merkleLeft + str(node.val) + merkleRight
            ~~node.merkle = _hash(toHash)~~
            return _hash(toHash)
```

# Why do we care?

I mentioned earlier Merkle Trees are useful because they allow us to verify if two trees are equivalent in constant time. How do we get this property, and why would we use it?

### Hashing

A good hash function provides several useful properties which give us confidence in the performance, validity, and practical usage of Merkle trees.

- **Deterministic** - The function will produce the same value every time, given the same input.
    - Computing a merkle root of two trees should ******always****** give the same result for each tree. Otherwise, hashing two equivalent trees might produce a different result for one, which would fail an equality check.
- **Efficient** - The function is computationally efficient.
    - A large part of what makes Merkle Trees attractive is their efficient time and space complexity (discussed below) compared to iterating through and comparing each node in each tree, or another naive method. Hashing typically occurs in constant time (or rather, linear in the size of the data hashed) - if not, another method to verify tree equivalence might prove more efficient.
- **Cannot be reversed** - Given the output of a hash function, a user cannot determine the input value.
    - One usage of Merkle Trees is to verify a set of transactions (a block) in a P2P network. If one could reconstruct these transactions given the root hash of a merkle tree any device in the network could track user behavior. This removes any guarantees of anonymity or privacy that a decentralized ledger may offer, and likely opens attack vectors to malicious parties.
- **Collision-resistant** - A hash function should produce the same hash for different values as little as possible (ideally never).
    - This means any two merkle trees have some probability of converging on the same root hash value. We can reduce this likelihood with a well-distributed hash function, but this probability will always be nonzero. Therefore it’s important to have some mechanism for resolution when two different values produce the same hash - otherwise we cannot trust hash equality.

These hashing properties allow us to be reasonably confident that our Merkle tree hash is ********************************************************************************************************************************consistent, efficient, secure, and accurately describes the tree********************************************************************************************************************************.

### Time and Space Complexity

Let’s revisit the example above to compute a Merkle Tree.

```python
def merkleHash(node):
            if not node:
                return '#'
            merkleLeft = merkleHash(node.left)
            merkleRight = merkleHash(node.right)
            toHash = merkleLeft + str(node.val) + merkleRight
            return _hash(toHash)
```

In this code we visit every node in the left subtree, then every node in the right subtree, then the current node. For each node we compute the hash of the left subtree, plus the current node, plus the hash of the right subtree. Since we have the hashes of left and right already, and we know our hash function is efficient, this happens in constant time. *O(n) * O(1) = O(n*) time complexity, where *n* is the number of nodes in the tree.

For our space complexity, the code above computes the Merkle root recursively. It will allocate a new stackframe for each recursive call, using *O(k)* memory where *k* is the maximum depth of the tree (worst case, all tree leaves are *k* deep and each recursive call allocates *k* stackframes). If we choose to compute a new tree that holds the Merkle hash for each node, we use *O(n)* auxiliary space - if we do this in-place and hold a variable for each child hash, we end up using constant space. *O(k) * O(1) = O(k)* space complexity for in-place, *O(k) * O(n) = O(nk)* space complexity to create a new tree.

Ok, that seems… fine. But this is a precomputation. Now that we have the hash value, we can just compare that Merkle root to the root of another tree - that’s *O(1)* time complexity. The alternative is walking through each tree and verifying each node matches the other - Potentially *O(n*) time. Consider verifying the same tree hundreds of times. Your time complexity is now only the cost you paid to compute the merkle root upfront. Even better, we no longer need to pass around entire trees to verify equality. Now we just send the hash value. This means we also only use constant extra space, after our precomputation.

# Applications of Merkle Trees

### Database Synchronization and Fault Tolerance

- Distributed databases like Cassandra and DynamoDB use Merkle Trees to sync replicas
    - A keyspace is divided into buckets. The data in each bucket is hashed, forming a leaf node, then those hashes are hashed themselves. The process continues until a full Merkle tree is built.
    - Each Merkle root is passed to replicas and compared with their local Merkle root. If their hashes disagree, the left root then the right root are compared to determine where the inconsistency lies.
        - ************************************************The amount of data to synchronize between replicas is now proportional to the differences, not the size of the data.************************************************
- Databases can use their synced replicas to recover from permanent failures in their node network.
    - If we sync our replicas often enough, we can easily recover from a permanent node failure by redirecting read queries to replicas.
    - Merkle trees provide this efficiency in asynchronous data synchronization.
        - **This results in high availability and a high degree of fault tolerance in distributed databases**.

### Cryptocurrency

- Cryptocurrencies powered by blockchain use merkle trees to validate transactions in a block.
    - Each block contains a header holding a hash, this hash is generated using a number of values including the merkle root of all transactions in that block and the previous block id.
    - This makes it near impossible to sneak in a modification to the transactions in the block - if you change a transaction the merkle root changes, which changes the previous block id of the next node in the blockchain, and so on.
        - **A change in a single transaction propagates through the entire system.**
- Blocks in the chain do not need to remember every transaction, only the merkle root, saving terabytes of space in the blockchain.
    - This also reduces network traffic and congestion between nodes in the network by an order of magnitude, allowing for greater efficiency and performance when verifying blocks.

### Version Control Systems

- Git uses Merkle Trees to efficiently check if
    - Git maintains history by storing a table of **********hash, blob********** where a blob is a file-like object and a hash is the hash of it’s contents. These hashes are built up on each other to form a Merkle tree which reflects the current state of the repo/commit.
    - When creating other versions of a repo or commit Git only cares about changed files - by checking the Merkle tree of the original and the Merkle tree of the new version, Git can determine which files were changed where the blob hashes differ.
        - ******************Git then only needs to store the contents of the changed files for each version. The unchanged files point to previous commits.******************