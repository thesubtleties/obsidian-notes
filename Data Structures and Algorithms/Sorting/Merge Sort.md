**Time and Space Complexity of Merge Sort**

- **Time Complexity**:
  - **Best Case**: O(n log n)
  - **Average Case**: O(n log n)
  - **Worst Case**: O(n log n)

- **Space Complexity**: O(n) - Merge sort requires additional space proportional to the size of the input array because of the recursive calls and the temporary arrays used during merging.

**Steps to Implement Merge Sort**

1. **Base Case**
   - If the array has one or zero elements, it is already sorted.
   - Return the array as is.

2. **Split the Array**
   - Find the middle index of the array.
   - Divide the array into two halves using the `slice` method.

3. **Recursive Sort**
   - Recursively sort both halves of the array by calling `mergeSort` on each half.

4. **Merge the Sorted Halves**
   - Use a helper function `merge` to combine the two sorted halves into a single sorted array.

**Example Code**

```javascript
function mergeSort(arr) {
    // Base case: if the array has one or zero elements, it is already sorted
    if (arr.length <= 1) return arr;

    // Find the middle index
    const middle = Math.floor(arr.length / 2);

    // Divide the array into two halves
    const half1 = arr.slice(0, middle);
    const half2 = arr.slice(middle);

    // Recursively sort both halves and merge them
    return merge(mergeSort(half1), mergeSort(half2));
}

function merge(arr1, arr2) {
    const res = [];
    let pointer1 = 0;
    let pointer2 = 0;

    // Merge the two arrays while both have elements
    while (pointer1 < arr1.length && pointer2 < arr2.length) {
        if (arr1[pointer1] <= arr2[pointer2]) {
            res.push(arr1[pointer1]);
            pointer1++;
        } else {
            res.push(arr2[pointer2]);
            pointer2++;
        }
    }

    // Add remaining elements from arr1 (if any)
    while (pointer1 < arr1.length) {
        res.push(arr1[pointer1]);
        pointer1++;
    }

    // Add remaining elements from arr2 (if any)
    while (pointer2 < arr2.length) {
        res.push(arr2[pointer2]);
        pointer2++;
    }

    return res;
}
```


**Key Points to Remember**

- **Divide and Conquer**: Merge sort uses the divide-and-conquer strategy to split the array into smaller subarrays and then merges them back together in sorted order.
- **Stable Sort**: Merge sort is a stable sort, meaning it preserves the relative order of equal elements.
- **Not In-Place**: Merge sort is not an in-place sorting algorithm as it requires additional space for the temporary arrays.

