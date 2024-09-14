
**Time and Space Complexity of Insertion Sort**

- **Time Complexity**:
  - **Best Case**: O(n) - Occurs when the array is already sorted.
  - **Average Case**: O(n^2) - Occurs when elements are in random order.
  - **Worst Case**: O(n^2) - Occurs when the array is sorted in reverse order.

- **Space Complexity**: O(1) - Insertion sort is an in-place sorting algorithm, meaning it requires a constant amount of additional memory space.

---

**Steps to Implement Insertion Sort**

1. **Outer Loop (For Loop)**
   - Iterate from the second element to the end of the array.
   - Syntax: `for (let i = 1; i < arr.length; i++) { ... }`

2. **Store the Current Element**
   - Save the element at the current index `i` to be inserted later.
   - Syntax: `let item = arr[i];`

3. **Inner Loop (While Loop)**
   - Initialize a variable to track the sorted section, starting from `i - 1`.
   - Syntax: `let sortedIndex = i - 1;`
   - Iterate towards the beginning of the array to find the correct position for the `item`.

4. **Shift Elements**
   - Compare each element in the sorted section with `item`.
   - If an element is larger than `item`, move it one slot to the right.
   - Syntax: 
     ```javascript
     while (sortedIndex >= 0 && arr[sortedIndex] > item) {
         arr[sortedIndex + 1] = arr[sortedIndex];
         sortedIndex--;
     }
     ```

5. **Insert the Stored Element**
   - Once the correct position is found (either at the start of the array or just after a smaller element), insert the `item`.
   - Syntax: `arr[sortedIndex + 1] = item;`

**Example Code**

```javascript
function insertionSort(arr) {
    for (let i = 1; i < arr.length; i++) {
        let item = arr[i];
        let sortedIndex = i - 1;
        while (sortedIndex >= 0 && arr[sortedIndex] > item) {
            arr[sortedIndex + 1] = arr[sortedIndex];
            sortedIndex--;
        }
        arr[sortedIndex + 1] = item;
    }
    return arr;
}
```

**Key Points to Remember**
- The outer loop iterates through each element starting from the second one.
- The inner loop shifts elements of the sorted section to the right.
- The stored `item` is inserted in its correct position within the sorted section.
- Be careful not to replace the `i` index value during the shifting process; only shift elements within the sorted section.


---
