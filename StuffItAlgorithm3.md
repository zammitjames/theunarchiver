# Description #

This algorithm simply uses Huffman encoding to encode individual bytes. The data begins with a header specifying the Huffman tree, followed by a bit stream of Huffman codes. All values are read most significant bit first, starting from the most significant bit of each consecutive byte.

# Header #

The bit stream begins with a header encoding the tree of Huffman codes. To read it, read the root node using this node-reading algorithm:

  * Read a single bit from the input stream.
  * If this bit is 1, the node being read is a leaf node. Read 8 bits to find the byte value of this leaf.
  * If the bit is 0, the node being read is a branch node. Read the 0-branch node followed by the 1-branch node.

# Decoding #

To read a byte from the input stream, start at the top node of the Huffman tree. Read a bit, and take the 0 or 1 branch of the tree depending. When you end up at a leaf node, you have the byte value. Do this for all bytes in the stream.