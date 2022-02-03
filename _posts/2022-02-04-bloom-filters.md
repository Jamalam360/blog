## Bloom Filters

Hello again...~~this is my third day running writing a post~~ this got split over two days actually! I'm finding it very enjoyable to be able to just say what I'm thinking about. I'm going to try to make a post that is a bit more in-depth and technical today, as that will help me understand the concept better myself. With that said, there may be some technical inaccuracies, but I'm going to try to make everything as clear as possible according to my understanding of Bloom filters.

I learnt about Bloom filters yesterday and I've been casually reading the Wikipedia article about them, I've made a simple implementation in TypeScript, which I'll show off later in this post.

### Bloom filter? What's that?

According to Wikipedia, a Bloom filter is:

> a space-efficient probabilistic data structure, conceived by Burton Howard Bloom in 1970, that is used to test whether an element is a member of a set.

So what does this mean? Here's my understanding:

- space-efficient: a Bloom filter tests whether an element is in a set without storing the element itself. In theory, a Bloom filter can store an infinite number of elements I believe, but there's a threshold after which all checks yield a positive result (more on how this can happen next).
- probabilistic: a Bloom filter doesn't return definite answers. As said by Wikipedia, false positives are possible, but false negatives are not. This effectively means the algorithm says whether an element is 'probably present' or 'definitely not present'

Let's take the example of checking whether a name is in a list of names. The following pseudocode example shows what the Bloom filter can do:

```
names <-- ["Jack", "John", "Jill", "Jane"]
filter <-- BloomFilter()

REPEAT name IN names
    filter.add(name)
ENDREPEAT

OUTPUT filter.contains("Jack") // true, set probably contains "Jack"
OUTPUT filter.contains("Bill") // false, set definitely does not contain "Bill"
```

So that's what a Bloom filter is for, now let's see how it works.

### What magic is this?

The simplest variant of a Bloom filter is simply a bit array filled with...well, bits! Each bit is initially set to 0:

```
[0, 0, 0, 0, 0, 0, 0, 0]
```

This Bloom filter is a bit useless (pun not intended) because of it's small size, but it's just an example.

When we add an element to the filter, we use multiple hash functions on it, then set the bits at those indexes to 1. In this scenario, a hash function takes in a string and returns a number (these values are entirely random and don't reflect any real hashes):

```
hash1("Jack") // 6
hash2("Jack") // 3
hash3("Jack") // 1
```

We then take these hashes and turn the bits at those indexes to 1:

```
[0, 1, 0, 1, 0, 0, 1, 0]
```

Now let's say we also want to insert Jill:

```
hash1("Jill") // 1
hash2("Jill") // 5
hash3("Jill") // 7
```

Now our bit array looks like this:

```
[0, 1, 0, 1, 0, 1, 1, 1]
```

Can you see why collisions/false positives are possible with Bloom filters? Multiple elements can have overlapping hashes. This is why we need to make sure we use an appropriate number of hash functions and bit array size (more on this later, in my TypeScript implementation).

I hear you asking:

> OK, so that's all well and good, but how do we know if an element is in the set?

Well, it's fairly simple. We hash the element using our same hash functions (this time we're checking whether Josh is in the set):

```
hash1("Josh") // 0
hash2("Josh") // 2
hash3("Josh") // 7
```

We now check the bits at those indexes (highlighted for your reading convenience):

```
[*0*, 1, *0*, 1, 0, 1, 1, *1*]
```

We can see that not all the bits are set to 1, so the element is definitely not in the set. But what if we had a false positive? Let's check if Bob is in our set:

```
hash1("Bob") // 1
hash2("Bob") // 6
hash3("Bob") // 7
```

Now let's check:

```
[0, *1*, 0, 1, 0, 1, *1*, *1*]
```

So yes, Bob is in the set! _Checks notes_... wait! No, Bob is not in the set! We can see that a false positive occurred because Bob's indicies have been set by other elements.

To control the number of false positives, we need to control the number of hash functions we use, as well as the size of the bit array. That brings us onto our next topic, to the suprise of no one:

### A Basic TypeScript Implementation

I'm have created a basic TypeScript implementation of a Bloom filter using the Deno runtime. The algorithm is based on [this article](https://www.geeksforgeeks.org/bloom-filters-introduction-and-python-implementation/) by GeeksForGeeks, which targets Python.

> If you want to use a Bloom filter in your own project, I would recommend using a pre-existing solution like [/x/bloomfilter](https://deno.land/x/bloomfilter@v2.3.0). My solution is purely created for learning purposes and I cannot guarantee that it will work for all cases.

With that said, the source code can be found [here](https://github.com/Jamalam360/bloom).

```ts
class BloomFilter {
  fpProbability: number;
  size: number;
  hashCount: number;
  bitArray: number[];

  constructor(items_count: number, fpProbability: number) {
    this.fpProbability = fpProbability;
    this.size = BloomFilter.getSize(items_count, fpProbability);
    this.hashCount = BloomFilter.getHashCount(this.size, items_count);
    this.bitArray = Array(this.size).fill(0);
  }

  public add(item: string): void {
    const digests = [];

    for (let i = 0; i < this.hashCount; i++) {
      const digest = hash(item, i) % this.size;
      digests.push(digest);
      this.bitArray[digest] = 1;
    }
  }

  public addAll(items: string[]): void {
    items.forEach((i) => this.add(i));
  }

  public check(item: string): boolean {
    for (let i = 0; i < this.hashCount; i++) {
      const digest = hash(item, i) % this.size;
      if (this.bitArray[digest] == 0) {
        return false;
      }
    }
    return true;
  }

  public static getSize(n: number, p: number): number {
    return Math.trunc(-(n * Math.log(p)) / Math.log(2) ** 2);
  }

  public static getHashCount(m: number, n: number): number {
    return Math.trunc((m / n) * Math.log(2));
  }

  public toBitString(): string {
    return this.bitArray.toString();
  }
}
```

This is my simple Bloom filter class. Let's look through it step by step:

```ts
fpProbability: number
```
This value represents the desired fale positive probability.

```ts
size: number
```
This value represents the size of the bit array. It is initialised using the mathematical formula:
`-(item_count * log(fp_probability)) / log(2) ** 2`
Now, maths like this is most definitely not my area of expertise, so I recommend you refer back to the GeeksForGeeks article for more information.

```ts
hashCount: number
```
This value represents the number of hash functions we use. It is initialised using the mathematical formula:
`(size / item_count) * log(2)`
Once again, I recommend you refer back to the GeeksForGeeks article for more information.

```ts
bitArray: number[]
```
This is the bit array that we use to store the data.

We can then see that when adding an element, we use multiple hash functions on it, then set the bits at those indexes to 1. To obtain these multiple hash functions, we use an implementation of MurmurHash3, which you can find in the source code linked above. We make each 'different' hash function by using a different seed.

### Wrapping Up

So that's about it for the basics of a Bloom filter. I hope you enjoyed this article, even though it probably has a few technical inaccuracies. I'm writing this for my own interest, rather than as a tutorial. I've found learning about Bloom filters very interesting because of the applications of such a data structure because of it's space-efficiency. I may revisit this later and find out just how much more space efficient it is with some example data.

Thanks for reading, goodbye for now - Jamalam