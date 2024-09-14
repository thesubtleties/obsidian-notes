
**Time and Space Complexity of Quick Sort**

- **Time Complexity**:
  - **Best Case**: O(n log n)
  - **Average Case**: O(n log n)
  - **Worst Case**: O(n^2) (Occurs when the pivot elements are the smallest or largest)

- **Space Complexity**:
  - **In-Place Version**: O(log n) - Due to the recursion stack.
  - **Non In-Place Version**: O(n) - Due to additional arrays used for partitioning.

**Steps to Implement Quick Sort (In-Place Version)**

1. **Base Case**
   - If the array has one or zero elements, it is already sorted.
   - Return the array as is.

2. **Choose a Pivot**
   - Typically, the last element is chosen as the pivot.

3. **Partition the Array**
   - Rearrange elements so that all elements less than the pivot come before it, and all elements greater come after it.

4. **Recursive Sort**
   - Recursively sort the subarrays before and after the pivot.

**In-Place Quick Sort Example Code (Lomuto Partition Scheme)**

```javascript
function quickSort(arr, low = 0, high = arr.length - 1) {
    if (low < high) {
        const pivotIndex = partition(arr, low, high);
        quickSort(arr, low, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, high);
    }
    return arr;
}

function partition(arr, low, high) {
    const pivot = arr[high];
    let i = low - 1;

    for (let j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            [arr[i], arr[j]] = [arr[j], arr[i]];
        }
    }

    [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
    return i + 1;
}
```

**Non In-Place Quick Sort Example Code**

```javascript
function quickSort(arr) {
    if (arr.length <= 1) return arr;

    const pivot = arr[arr.length - 1];
    let left = [];
    let right = [];

    for (let i = 0; i < arr.length - 1; i++) {
        if (arr[i] < pivot) {
            left.push(arr[i]);
        } else {
            right.push(arr[i]);
        }
    }

    return [...quickSort(left), pivot, ...quickSort(right)];
}
```

**Differences Between In-Place and Non In-Place Quick Sort**

- **In-Place Quick Sort**:
  - **Memory Efficient**: Uses the original array for sorting, requiring less additional memory.
  - **Performance**: Typically faster due to reduced memory overhead.
  - **Complexity**: Slightly more complex to implement correctly due to in-place partitioning and swaps.

- **Non In-Place Quick Sort**:
  - **Simpler Logic**: Easier to understand and implement since it uses additional arrays for partitioning.
  - **Memory Usage**: Requires extra memory for the additional arrays, which can be significant for large arrays.
  - **Use Case**: Suitable for educational purposes and understanding the quick sort logic.

**Hoare Partition Scheme**

**Steps to Implement Quick Sort with Hoare Partition Scheme**

1. **Base Case**
   - If the array has one or zero elements, it is already sorted.
   - Return the array as is.

2. **Choose a Pivot**
   - Typically, the middle element is chosen as the pivot.

3. **Partition the Array**
   - Use two pointers to rearrange elements so that all elements less than the pivot come before it, and all elements greater come after it.

4. **Recursive Sort**
   - Recursively sort the subarrays before and after the pivot.

**Hoare Partition Quick Sort Example Code**

```javascript
function quickSort(arr, low = 0, high = arr.length - 1) {
    if (low < high) {
        const pivotIndex = partition(arr, low, high);
        quickSort(arr, low, pivotIndex);
        quickSort(arr, pivotIndex + 1, high);
    }
    return arr;
}

function partition(arr, low, high) {
    const pivot = arr[Math.floor((low + high) / 2)];
    let left = low;
    let right = high;

    while (true) {
        while (arr[left] < pivot) {
            left++;
        }
        while (arr[right] > pivot) {
            right--;
        }
        if (left >= right) {
            return right;
        }
        [arr[left], arr[right]] = [arr[right], arr[left]];
        left++;
        right--;
    }
}
```

**Key Points for Hoare Partition Scheme**

- **Efficiency**: Generally more efficient than the Lomuto partition scheme as it minimizes the number of swaps.
- **Balanced Partitions**: Tends to produce more balanced partitions, especially for arrays with repeated elements or partially sorted arrays.
- **Use Case**: Ideal for scenarios where performance is critical and the array size is large.

