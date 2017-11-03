# Test page

* Point 1
* Point 2
* Point 3

```python
x = tf.placeholder(tf.float32, shape=(2, 2))
y = tf.matmul(x, x)

with tf.Session() as sess:
  print(sess.run(y))  # ERROR: will fail because x was not fed.

  rand_array = np.random.rand(2, 2) # we should get data from some training data
  print(sess.run(y, feed_dict={x: rand_array}))  # Will succeed.
```