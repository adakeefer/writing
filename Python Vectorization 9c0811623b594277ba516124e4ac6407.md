# Python Vectorization

Created: October 20, 2023 9:34 AM
Tags: CS

- say you have a list up to 1 million and you want to multiply everything by 3
- The order in which you do the operations on the vector doesn’t matter
- Modern computers have multiple cores meaning parallel processing. We can divide a list into quadrents and have the computer work on it simultaneously
- Main idea: **************take advantage of computers architecture to speed up computation**************
- numpy speed up:
    - in python you have lists of whatever. In numpy you declare the type up front
    - numpy has locality. stores your vectors in the same area of your memory. not necessarily the case in native python.
- ************************************Main idea: avoid loops as much as possible, use array expressions instead************************************
- examples:
- 

```python
import numpy as np
from time import time
n = 10000000
#make random numbers
#fill two arrays
#add them for numpy arrays this is just a + b
# startTime = time()
#endTime = time()
# this code took blah seconds
#verify the two arrays are the same
```