# Linked List Bubble Sort

This document provides documentation for the `bubble_sort_linked_list` function and the `ListNode` class.

## ListNode Class

The `ListNode` class represents a node in a singly linked list.

### Attributes

- `val`: The value of the node.
- `next`: The next node in the list.

## bubble_sort_linked_list Function

The `bubble_sort_linked_list` function sorts a singly linked list in ascending order using the bubble sort algorithm.

### Parameters

- `head` (Optional[ListNode]): The head node of the singly linked list.

### Returns

- `Optional[ListNode]`: The head node of the sorted linked list.

### Example

```python
# Demonstrate the Linked List Bubble Sort
print("\n--- Linked List Bubble Sort ---")
unsorted_list = ListNode(64, ListNode(34, ListNode(25, ListNode(12, ListNode(22, ListNode(11, ListNode(910)))))))
print(f"Unsorted Linked List: {unsorted_list}")
sorted_list = bubble_sort_linked_list(unsorted_list)
print(f"Sorted Linked List:   {sorted_list}")
```
