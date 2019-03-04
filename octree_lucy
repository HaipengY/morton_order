from mpl_toolkits.mplot3d import Axes3D
import math
import struct
import matplotlib.pyplot as plt


tree = [[]for i in range(20)]
level_of_tree = 20


def build_tree(point, depth):
    if depth < level_of_tree:
        bit_level = depth - 3
        if len(tree[depth]) == 0:
            tree[depth].append(point)
            return
        else:
            diff_bit = [0, 0, 0]
            for i in range(3):
                diff_bit[i] = msbdiff(point[i] + 640, tree[depth][0][i] + 640)
            if max(diff_bit) == bit_level:
                tree[depth].append(point)
                return
            elif max(diff_bit) > bit_level:
                write_to_file(depth)
                new_point = merge(depth)
                build_tree(new_point, depth+1)
                tree[depth].clear()
                tree[depth].append(point)
                return


def merge(depth):
    point_merge = [0, 0, 0]
    for i in range(3):
        s = 0
        for j in range(len(tree[depth])):
            s = s + tree[depth][j][i]
            point_merge[i] = s/len(tree[depth])
    return point_merge


def write_to_file(depth):
    binary_format = '>3f'
    if depth < level_of_tree:
        for i in range(len(tree[depth])):
            tree_file[depth].write(struct.pack(binary_format, tree[depth][i][0], tree[depth][i][1], tree[depth][i][2]))
    return


def msbdiff(a, b):
    a_m, a_e = math.frexp(a)
    b_m, b_e = math.frexp(b)
    a_m = int(a_m * 2**23)
    b_m = int(b_m * 2**23)
    # print(a, b, a_m, b_m, a_m ^ b_m, a_e, b_e)
    if a_e == b_e:
        if a_m == b_m:
            bit_different = 23
        else:
            # print(a_m ^ b_m)
            bit_different = int(math.log2(a_m ^ b_m) + 1) + a_e - 23
        return bit_different
    else:
        return max(a_e, b_e)


'''
def msbdiff(a, b):
    a_m, a_e = math.frexp(a)
    b_m, b_e = math.frexp(b)
    a_m = int(a_m * 2 ** 24)
    b_m = int(b_m * 2 ** 24)
    # print(a, b, a_m, b_m, a_m ^ b_m, a_e, b_e)
    if a_e == b_e:
        if a_m == b_m:
            bit_different = 24
        else:
            # print(a_m ^ b_m)
            bit_different = 24 - int(math.log2(a_m ^ b_m) + 1)
        return a_e - bit_different - 1
    else:
        return max(a_e, b_e) - 1
'''


def morton_compare(p, q):
    assert len(p) == len(q)
    d = len(p)
    # print('the coordinate of two points are:', p, q)
    x, dim = (-128, 0)
    for j in range(d):
        y = msbdiff(p[j] + 640, q[j] + 640)
        # print('msbd[', j, ']is', y)
        if x < y:
            x, dim = (y, j)
    # print('dimension with biggest difference is', dim)
    # print('values :', p[dim], q[dim])
    return p[dim] < q[dim]


def read_point():
    point_list = []
    with open('lucy_original.ply', 'rb') as f:
        fm = '>3f'
        f.seek(180, 1)
        for x in range(14027872):
            chuck = f.read(struct.calcsize(fm))
            element = struct.unpack(fm, chuck)
            point_list.append(element)
        return point_list


class MinHeap:
    def __init__(self):
        self.list = [(0, 0, 0)]
        self.size = 0

    def up(self, i):
        while i // 2 > 0:
            if morton_compare(self.list[i], self.list[i // 2]):
                self.list[i], self.list[i // 2] = self.list[i // 2], self.list[i]
            i = i // 2

    def insert(self, k):
        self.list.append(k)
        self.size += 1
        self.up(self.size)

    def min_child(self, i):
        if i * 2 + 1 > self.size:
            return i * 2
        else:
            if morton_compare(self.list[i * 2], self.list[i * 2 + 1]):
                return i * 2
            else:
                return i * 2 + 1

    def extract_min(self):
        if self.size > 0:
            top_point = self.list[1]
            self.list[1], self.list[self.size] = self.list[self.size], self.list[1]
            self.list.pop()
            self.size -= 1
            self.down(1)
            return top_point

    def down(self, i):
        while i * 2 <= self.size:
            minc = self.min_child(i)
            # print(i, minc)
            if morton_compare(self.list[minc], self.list[i]):
                self.list[i], self.list[minc] = self.list[minc], self.list[i]
            i = minc

    def build(self, point_tuple):
        i = len(point_tuple) // 2
        self.size = len(point_tuple)
        self.list.extend(point_tuple)
        print(len(self.list), self.size, i)
        while i > 0:
            self.down(i)
            i -= 1


tree_file = []
for i in range(level_of_tree):
    file = open('lucy_' + str(i), 'wb')
    tree_file.append(file)
point_file = open('ordered_point', 'w')
point_heap = MinHeap()
point_heap.build(read_point())


fig = plt.figure()
ax = Axes3D(fig)
c_x = []
c_y = []
c_z = []


for x in range(14027872):
    min_in_heap = list(point_heap.extract_min())
    # point_file.write(str(min_in_heap))
    build_tree(min_in_heap, 0)
    if x % 2000 == 0:
        c_x.append(min_in_heap[0])
        c_y.append(min_in_heap[1])
        c_z.append(min_in_heap[2])
ax.scatter(c_x, c_y, c_z, c='r', marker='o')
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
plt.show()
