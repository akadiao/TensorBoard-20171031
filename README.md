# #!/usr/bin/python
# # coding:utf-8

import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt

# add one more layer and return the output of this layer
def add_layer(inputs, in_size, out_size, n_layer, activation_function=None):
    layer_name = 'layer%s' % n_layer
    with tf.name_scope('layer'):
        with tf.name_scope('weights'):
            Weights = tf.Variable(tf.random_normal([in_size, out_size]), name='W')
            tf.summary.histogram(layer_name + '/weights', Weights)
        with tf.name_scope('biases'):
            biases = tf.Variable(tf.zeros([1,out_size]) + 0.1, name='b')
            tf.summary.histogram(layer_name + '/biases', biases)
        with tf.name_scope('Wx_plus_b'):
            Wx_plus_b = tf.add(tf.matmul(inputs, Weights), biases, name='Wpb')
        if activation_function is None:
            outputs = Wx_plus_b
        else:
            outputs = activation_function(Wx_plus_b)
        tf.summary.histogram(layer_name + '/outputs', outputs)
        return outputs

#define placeholder for inputs
with tf.name_scope('inputs'):
    xs = tf.placeholder(tf.float32, [None, 1], name='x_input')
    ys = tf.placeholder(tf.float32, [None, 1], name='y_input')

# make up some real data
x_data = np.linspace(-1.0, 1.0, 300, dtype=np.float32)[:, np.newaxis]
noise = np.random.normal(0.0, 0.05, x_data.shape)
y_data = np.square(x_data) + 0.5 + noise

# add hidden layer and output layer
l1 = add_layer(xs, 1, 10, n_layer=1, activation_function=tf.nn.relu)
prediction = add_layer(l1, 10, 1, n_layer=2, activation_function=None)

# the error between prediction and real data
with tf.name_scope('loss'):
    loss = tf.reduce_mean(tf.reduce_sum(tf.square(ys - prediction), reduction_indices=[1]), name='L')
    tf.summary.scalar('loss', loss)
with tf.name_scope('train'):
    train_step = tf.train.GradientDescentOptimizer(0.1).minimize(loss)

init = tf.initialize_all_variables()
with tf.Session() as sess:
    sess.run(init)
    fig = plt.figure()
    ax = fig.add_subplot(1, 1, 1)
    ax.scatter(x_data, y_data)
    plt.show(block=False)
    merged =tf.summary.merge_all()
    writer = tf.summary.FileWriter('/tmp/tensorlogs/ex5', sess.graph)
    for i in range(1000):
        sess.run(train_step, feed_dict={xs: x_data, ys: y_data})
        if i % 50 == 0:
            # print(i, sess.run(loss, feed_dict = {xs : x_data, ys : y_data}))
            try:
                ax.lines.remove(lines[0])
            except Exception:
                pass
            result, prediction_value = sess.run([merged, prediction], feed_dict={xs: x_data, ys: y_data})
            lines = ax.plot(x_data, prediction_value, 'r-', lw=5)
            plt.pause(0.1)
            writer.add_summary(result, i)



#  tensorboard --logdir='/tmp/tensorlogs/ex5'
