segm-lstm
===

- description
  - string segmentation(auto-spacing) using LSTM(tensorflow)
    - input
      - string, ex) '이것을띄어쓰기하면어떻게될까요'
    - output
      - string, ex) '이것을 띄어쓰기하면 어떻게 될까요' 
  - model
    - x : '이것을 띄어쓰기하면 어떻게 될까요'
	- y : '0 0 1 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0'
	  - 1 : if next char is space
	  - 0 : if next char is not space
    - learn to predict tag sequence

- sketch code
```
$ python sketch.py
...
step : 970,cost : 0.0117462
step : 980,cost : 0.0115485
step : 990,cost : 0.0113553
out = 이것을 띄어쓰기하면 어떻게 될까요
out = 아버지가 방에 들어가신다.
```

- how to deal variable-length input
```
let's try to use sliding window method and early stop.

n_steps = 30

- training
  if len(sentence) >= 1 and len(sentence) < n_steps : padding with '\t'
  if len(sentence) > n_steps : move next batch pointer(sliding window)

- inference
  if len(sentence) >= 1 and len(sentence) < n_steps : padding with '\t'
  if len(sentence) > n_steps : 
    move next batch pointer(sliding window)
	merge result into one array
	decoding
```

- train
```
$ python train.py --train=train.txt --validation=validation.txt --model=model

$ python train.py --train=big.txt --validation=validation.txt --model=model
...
7 th sentence ... done
8 th sentence ... done
9 th sentence ... done
seq : 29,validation cost : 124.562777519,validation accuracy : 0.942500010133
save dic
save model(final)
end of training
```

- inference
```
$ python inference.py --model=model < test.txt
...
model restored from model/segm.ckpt
out = 이것을 띄어 쓰기하면 어 떻게 될까요.
out = 아버지가 방에 들어 가신다.
out = SK이노베이션, GS, S-Oil, 대림산업, 현대중공업 등 대규모 적자를 내던
out = 기업들이 극한 구조조정을 통해 흑자로 전환하거나
out = 적자폭을 축소한 것이영 업이익 개선을 이끈 것으로 풀이된다.

$ python inference.py --model=model < test.txt
...
model restored from model/segm.ckpt
out = 이것 을 띄어쓰기 하면어 떻게 될 까 요.
out = 아버 지 가방에들 어가 신다 .
out = SK이노베이 션, GS , S -Oil, 대림산 업, 현대중공 업등 대규모적자 를 내 던
out = 기업들이 극 한 구조조정을 통해흑자로 전환하거나
out = 적자 폭을 축소한 것 이 영업이익 개선을 이 끈 것 으로 풀이 된 다.

# it seems that training data is not enough...
```

- character-based word2vec
```
# usage : https://github.com/tensorflow/tensorflow/tree/master/tensorflow/models/embedding
# for training non-ascii data
$ cd tensorflow/tensorflow/models/embedding
$ vi word2vec_optimized.py
  ...
  def save_vocab(self):
    """Save the vocabulary to a file so the model can be reloaded."""
    opts = self._options
    with open(os.path.join(opts.save_path, "vocab.txt"), "w") as f:
      for i in xrange(opts.vocab_size):
        f.write("%s %d\n" % (tf.compat.as_text(opts.vocab_words[i]).encode('utf-8'),
                             opts.vocab_counts[i]))
  ...
# preprocessing for character-based

# train word2vec
$ python word2vec_optimized.py --train_data=train.txt --eval_data=questions-words.txt --save_path=tmp

# test word2vec
$ cd segm-lstm
$ python test_word2vec.py --model_path=tmp
```

- development note
```
- training speed is very slow despite of using GPU. 
  how make it faster?
  : what about using word2vec(character-based)?
  using a pretrained word embedding
  : https://codedump.io/share/GsajBJMQJ50P/1/using-a-pre-trained-word-embedding-word2vec-or-glove-in-tensorflow
```
