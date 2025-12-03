import math
import time
import sys
import random


class BloomFilter:
    def __init__(self, n, p):
        """
        Initialize Bloom Filter.
        n = expected number of elements
        p = desired false positive probability
        """
        self.n = n
        self.p = p

        # Optimal values
        self.m = self._optimal_bit_array_size(n, p)
        self.k = self._optimal_hash_count(self.m, n)

        # Pure python bit array (list of 0/1)
        self.bit_array = [0] * self.m

        print(f"[INFO] Pure Python Bloom Filter initialized.")
        print(f"       Expected elements       : {n}")
        print(f"       Desired FP rate         : {p}")
        print(f"       Bit array size (m)      : {self.m}")
        print(f"       Number of hash funcs (k): {self.k}")
        print("--------------------------------------------------------")

    # --------------------------------------------------------
    # Mathematical calculations
    # --------------------------------------------------------

    def _optimal_bit_array_size(self, n, p):
        """
        m = -(n * ln(p)) / (ln(2)^2)
        """
        return int(-(n * math.log(p)) / (math.log(2) ** 2))

    def _optimal_hash_count(self, m, n):
        """
        k = (m/n) * ln(2)
        """
        return int((m / n) * math.log(2))

    # --------------------------------------------------------
    # Pure Python hash functions
    # --------------------------------------------------------

    def _hash(self, item, seed):
        """
        Fully pure Python hash function.
        Combines built-in hash with a seed.
        Ensures deterministic output.
        """
        h = hash(item)                # Python's built-in hash
        h ^= seed * 0x5bd1e995        # mix bits
        h = (h * 0xc6a4a7935bd1e995) & 0xFFFFFFFFFFFFFFFF
        return h % self.m

    # --------------------------------------------------------
    # Insert and lookup
    # --------------------------------------------------------

    def add(self, item):
        for i in range(self.k):
            pos = self._hash(item, i)
            self.bit_array[pos] = 1

    def check(self, item):
        for i in range(self.k):
            pos = self._hash(item, i)
            if self.bit_array[pos] == 0:
                return False
        return True  # Might exist

    # --------------------------------------------------------
    # Debug / stats
    # --------------------------------------------------------

    def memory_usage(self):
        """
        Each bit is stored as Python int → overhead is ~28 bytes per entry.
        """
        return sys.getsizeof(self.bit_array) + sum(sys.getsizeof(x) for x in self.bit_array)


# --------------------------------------------------------
#       Performance & Accuracy Test (Pure Python)
# --------------------------------------------------------

def test_pure_bloom_filter():
    n = 5000       # expected elements
    p = 0.01       # false positive target

    bf = BloomFilter(n, p)

    existing_items = [f"user_{i}" for i in range(n)]
    absent_items = [f"ghost_{i}" for i in range(n)]

    # Insert items
    start_time = time.time()
    for item in existing_items:
        bf.add(item)
    insertion_time = time.time() - start_time

    # Check for false positives
    false_pos = 0
    for item in absent_items:
        if bf.check(item):
            false_pos += 1

    fp_rate = false_pos / len(absent_items)
    mem_used = bf.memory_usage()

    print("\n================== PURE PYTHON TEST RESULTS ==================")
    print(f"Expected false positive rate  : {p}")
    print(f"Measured false positive rate  : {fp_rate:.4f}")
    print(f"Insertion time               : {insertion_time:.4f} sec")
    print(f"Memory used                  : {mem_used} bytes")
    print("================================================================")


if __name__ == "__main__":
    test_pure_bloom_filter()
