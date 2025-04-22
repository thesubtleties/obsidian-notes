## Definition 📝
Mathematics and geometry fundamentals are essential tools for solving programming problems, particularly in algorithm design, computer graphics, and computational geometry. This reference sheet covers key mathematical and geometric concepts commonly used in programming and algorithm development.

## Number Theory Fundamentals ➗

### GCD (Greatest Common Divisor) & LCM (Least Common Multiple)

**GCD**: The largest positive integer that divides two or more integers without a remainder.
**LCM**: The smallest positive integer that is divisible by two or more integers.

#### Implementation - Euclidean Algorithm for GCD
```python
# Recursive implementation
def gcd(a, b):
    if b == 0:
        return a
    return gcd(b, a % b)

# Iterative implementation
def gcd_iterative(a, b):
    while b:
        a, b = b, a % b
    return a

# Calculate LCM using GCD
def lcm(a, b):
    return abs(a * b) // gcd(a, b)
```

#### Complexity Analysis
- Time Complexity: O(log(min(a,b)))
- Space Complexity: O(log(min(a,b))) for recursive, O(1) for iterative

#### Applications
- Fraction simplification
- Cryptography (RSA algorithm)
- Modular arithmetic
- Finding cycles in sequences

## Prime Numbers 🔑

### Definition
A prime number is a natural number greater than 1 that is not a product of two smaller natural numbers.

### Prime Testing - Primality Test
```python
def is_prime(n):
    if n <= 1:
        return False
    if n <= 3:
        return True
    if n % 2 == 0 or n % 3 == 0:
        return False
    i = 5
    while i * i <= n:
        if n % i == 0 or n % (i + 2) == 0:
            return False
        i += 6
    return True
```

### Sieve of Eratosthenes (Finding all primes up to n)
```python
def sieve_of_eratosthenes(n):
    primes = [True for i in range(n+1)]
    p = 2
    while p * p <= n:
        if primes[p]:
            for i in range(p * p, n+1, p):
                primes[i] = False
        p += 1
    
    prime_list = []
    for p in range(2, n+1):
        if primes[p]:
            prime_list.append(p)
    return prime_list
```

#### Complexity Analysis
- Primality Test: O(√n)
- Sieve of Eratosthenes: O(n log log n)

#### Applications
- Cryptography (RSA, Diffie-Hellman)
- Hash functions
- Random number generation
- Number theory problems

## Factorial and Combinations 🧮

### Factorial
The product of all positive integers less than or equal to n.
n! = n × (n-1) × (n-2) × ... × 3 × 2 × 1

```python
# Recursive implementation
def factorial_recursive(n):
    if n == 0 or n == 1:
        return 1
    return n * factorial_recursive(n - 1)

# Iterative implementation
def factorial_iterative(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result
```

### Combinations
The number of ways to choose k items from n items without repetition and without order.
C(n,k) = n! / (k! × (n-k)!)

```python
def combination(n, k):
    # Optimize by using the smaller value of k or n-k
    k = min(k, n - k)
    result = 1
    for i in range(1, k + 1):
        result *= (n - (i - 1))
        result //= i
    return result
```

#### Complexity Analysis
- Factorial: O(n) time, O(n) space for recursive, O(1) space for iterative
- Combination: O(k) time, O(1) space

#### Applications
- Probability calculations
- Permutations and combinations problems
- Dynamic programming
- Binomial theorem

## Geometric Fundamentals 📐

### Points and Vectors
```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    # Vector addition
    def add(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    
    # Dot product
    def dot(self, other):
        return self.x * other.x + self.y * other.y
    
    # Cross product (magnitude in 2D)
    def cross(self, other):
        return self.x * other.y - self.y * other.x
    
    # Vector magnitude
    def magnitude(self):
        return (self.x**2 + self.y**2)**0.5
```

### Line Intersection ↗️

#### Line Representation
Lines can be represented in several ways:
1. Point-slope form: y - y₁ = m(x - x₁)
2. Slope-intercept form: y = mx + b
3. Two points: Line passing through (x₁, y₁) and (x₂, y₂)
4. Parametric form: P = P₁ + t(P₂ - P₁)

#### Line Segment Intersection Algorithm
```python
def orientation(p, q, r):
    """
    Returns the orientation of triplet (p, q, r).
    0 --> Collinear
    1 --> Clockwise
    2 --> Counterclockwise
    """
    val = (q.y - p.y) * (r.x - q.x) - (q.x - p.x) * (r.y - q.y)
    if val == 0:
        return 0
    return 1 if val > 0 else 2

def do_intersect(p1, q1, p2, q2):
    """
    Returns true if line segments p1q1 and p2q2 intersect.
    """
    # Find the four orientations needed for general and special cases
    o1 = orientation(p1, q1, p2)
    o2 = orientation(p1, q1, q2)
    o3 = orientation(p2, q2, p1)
    o4 = orientation(p2, q2, q1)

    # General case
    if o1 != o2 and o3 != o4:
        return True

    # Special Cases
    # p1, q1 and p2 are collinear and p2 lies on segment p1q1
    if o1 == 0 and on_segment(p1, p2, q1):
        return True

    # p1, q1 and q2 are collinear and q2 lies on segment p1q1
    if o2 == 0 and on_segment(p1, q2, q1):
        return True

    # p2, q2 and p1 are collinear and p1 lies on segment p2q2
    if o3 == 0 and on_segment(p2, p1, q2):
        return True

    # p2, q2 and q1 are collinear and q1 lies on segment p2q2
    if o4 == 0 and on_segment(p2, q1, q2):
        return True

    return False

def on_segment(p, q, r):
    """
    Returns true if point q lies on line segment 'pr'
    """
    return (q.x <= max(p.x, r.x) and q.x >= min(p.x, r.x) and
            q.y <= max(p.y, r.y) and q.y >= min(p.y, r.y))

def find_intersection_point(p1, q1, p2, q2):
    """
    Find the intersection point of two line segments
    """
    # Line 1 represented as a1x + b1y = c1
    a1 = q1.y - p1.y
    b1 = p1.x - q1.x
    c1 = a1 * p1.x + b1 * p1.y

    # Line 2 represented as a2x + b2y = c2
    a2 = q2.y - p2.y
    b2 = p2.x - q2.x
    c2 = a2 * p2.x + b2 * p2.y

    determinant = a1 * b2 - a2 * b1

    if determinant == 0:
        # Lines are parallel or coincident
        return None
    else:
        x = (b2 * c1 - b1 * c2) / determinant
        y = (a1 * c2 - a2 * c1) / determinant
        return Point(x, y)
```

#### Complexity Analysis
- Line Intersection Check: O(1)
- Finding Intersection Point: O(1)

## Polygon Operations 🔷

### Area of a Polygon (Shoelace formula)
```python
def polygon_area(vertices):
    """
    Calculate the area of a polygon using the Shoelace formula.
    vertices: list of Point objects representing the polygon vertices in order
    """
    n = len(vertices)
    area = 0.0
    
    for i in range(n):
        j = (i + 1) % n
        area += vertices[i].x * vertices[j].y
        area -= vertices[j].x * vertices[i].y
    
    area = abs(area) / 2.0
    return area
```

### Point in Polygon (Ray Casting Algorithm)
```python
def is_point_in_polygon(point, vertices):
    """
    Determine if a point is inside a polygon using ray casting algorithm.
    Returns True if the point is inside, False otherwise.
    """
    n = len(vertices)
    inside = False
    
    p1x, p1y = vertices[0].x, vertices[0].y
    for i in range(n + 1):
        p2x, p2y = vertices[i % n].x, vertices[i % n].y
        
        if point.y > min(p1y, p2y) and point.y <= max(p1y, p2y) and point.x <= max(p1x, p2x):
            if p1y != p2y:
                x_intersect = (point.y - p1y) * (p2x - p1x) / (p2y - p1y) + p1x
            
            if p1x == p2x or point.x <= x_intersect:
                inside = not inside
        
        p1x, p1y = p2x, p2y
    
    return inside
```

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Mathematical algorithms are often elegant and efficient
- Geometric algorithms provide solutions to real-world spatial problems
- Number theory concepts are fundamental to cryptography and security
- Many mathematical algorithms have guaranteed time and space complexity

### Disadvantages ❌
- Floating-point precision issues can cause errors in geometric calculations
- Some mathematical algorithms can be complex to implement correctly
- Naive implementations of number theory algorithms may be inefficient for large inputs
- Edge cases in geometric algorithms (collinearity, degeneracy) require careful handling

## Real-world Applications 🌐
- Computer graphics and game development
- Geographic Information Systems (GIS)
- Computational geometry in CAD/CAM systems
- Robotics and path planning
- Cryptography and security
- Physics simulations
- Data compression
- Machine learning and optimization

## Further Reading 📚

### Books
- "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
- "Computational Geometry: Algorithms and Applications" by de Berg, Cheong, van Kreveld, and Overmars
- "Concrete Mathematics" by Graham, Knuth, and Patashnik
- "A Programmer's Introduction to Mathematics" by Jeremy Kun

### Online Resources
- [Khan Academy - Mathematics](https://www.khanacademy.org/math)
- [GeeksforGeeks - Mathematical Algorithms](https://www.geeksforgeeks.org/mathematical-algorithms/)
- [Brilliant.org - Number Theory](https://brilliant.org/wiki/number-theory/)
- [Computational Geometry Algorithms Library (CGAL)](https://www.cgal.org/)

### Interactive Tools
- [GeoGebra](https://www.geogebra.org/) - Interactive geometry, algebra, and calculus
- [Desmos](https://www.desmos.com/calculator) - Graphing calculator
- [Visualgo](https://visualgo.net/) - Visualizing algorithms and data structures

---