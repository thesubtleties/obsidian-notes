**Time and Space Complexity of Bubble Sort**

- **Time Complexity**:
  - **Best Case**: O(n) - Occurs when the array is already sorted.
  - **Average Case**: O(n^2)
  - **Worst Case**: O(n^2) - Occurs when the array is sorted in reverse order.

- **Space Complexity**: O(1) - Bubble sort is an in-place sorting algorithm, meaning it requires a constant amount of additional memory.

**Steps to Implement Bubble Sort**

1. **Outer Loop**:
   - Iterate from the beginning of the array to the end.
   - Each pass through the array places the next largest element in its correct position.

2. **Inner Loop**:
   - Iterate from the beginning of the array to the current end of the unsorted section.
   - Compare each pair of adjacent elements and swap them if they are in the wrong order.

3. **Repeat**:
   - Continue iterating through the array until no more swaps are needed.

**Example Code**

```javascript
function bubbleSort(arr) {
    let n = arr.length;
    let swapped;

    do {
        swapped = false;
        for (let i = 1; i < n; i++) {
            if (arr[i - 1] > arr[i]) {
                // Swap arr[i - 1] and arr[i]
                [arr[i - 1], arr[i]] = [arr[i], arr[i - 1]];
                swapped = true;
            }
        }
        n--; // Reduce the range of elements to be sorted
    } while (swapped);

    return arr;
}
```

**Key Points to Remember**

- **Simplicity**: Bubble sort is one of the simplest sorting algorithms to understand and implement.
- **Inefficiency**: Despite its simplicity, bubble sort is inefficient for large datasets due to its O(n^2) time complexity.
- **Stable Sort**: Bubble sort is a stable sorting algorithm, meaning it preserves the relative order of equal elements.
- **In-Place Sorting**: Bubble sort sorts the array in place, requiring only a constant amount of additional memory.

