**Time and Space Complexity of Selection Sort**

- **Time Complexity**:
  - **Best Case**: O(n^2)
  - **Average Case**: O(n^2)
  - **Worst Case**: O(n^2)

- **Space Complexity**: O(1) - Selection sort is an in-place sorting algorithm, meaning it requires a constant amount of additional memory.

**Steps to Implement Selection Sort**

1. **Outer Loop**:
   - Iterate from the beginning to the end of the array.
   - Each pass through the array selects the smallest element from the unsorted section and swaps it with the first unsorted element.

2. **Find the Minimum**:
   - Use an inner loop to find the smallest element in the unsorted section of the array.

3. **Swap**:
   - Swap the smallest element found with the first unsorted element.

4. **Repeat**:
   - Move the boundary of the unsorted section one element to the right and repeat until the entire array is sorted.

**Example Code**

```javascript
function selectionSort(arr) {
    let n = arr.length;

    for (let i = 0; i < n - 1; i++) {
        // Assume the minimum is the first element
        let minIndex = i;

        // Find the minimum element in the unsorted part of the array
        for (let j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }

        // Swap the found minimum element with the first unsorted element
        if (minIndex !== i) {
            [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
        }
    }

    return arr;
}
```

**Key Points to Remember**

- **Simplicity**: Selection sort is simple and intuitive to understand and implement.
- **Inefficiency**: Despite its simplicity, selection sort is inefficient for large datasets due to its O(n^2) time complexity.
- **In-Place Sorting**: Selection sort sorts the array in place, requiring only a constant amount of additional memory.
- **Not Stable**: Selection sort is not a stable sorting algorithm, meaning it may not preserve the relative order of equal elements.

