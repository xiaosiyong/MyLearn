# Leetcode 

## 算法

### 1、三数之和 

给定一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？找出所有满足条件且不重复的三元组。[15. 三数之和](https://leetcode-cn.com/problems/3sum/)

注意：答案中不可以包含重复的三元组。

例如, 给定数组 nums = [-1, 0, 1, 2, -1, -4]，

满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]

解答：最暴力的解法，直接循环，然后判断是否存在，但时间复杂度为O($n^3$)，不满足要求。参考了一下别人的解法，思路如下：

1、先给数组排序，我用的快排，时间复杂度O(n*logN)；

2、拍完序之后，从0开始遍历，如果开始就大于0，直接退出

3、循环过程中，每遇到相同的元素，直接跳过，这样可以省略掉去重操作。

代码实现：

~~~go
func threeSum(nums []int)[][]int{
	var result [][]int
	if len(nums)<3 {
		return result
	}
	quickSort(nums,0,len(nums)-1)
	fmt.Println(nums)
	for i := 0;i < len(nums);i++ {
		if nums[i] > 0 {
			break
		}
		if i > 0 && nums[i] == nums[i-1] {//去重
			continue
		}
		l := i + 1
		h := len(nums)-1
		for l < h {
			sum := nums[i] + nums[l] + nums[h]
			if sum == 0 {
				var temp []int
				temp = append(temp,nums[i])
				temp = append(temp,nums[l])
				temp = append(temp,nums[h])
				result = append(result,temp)
				for l < h && nums[l] == nums[l+1]{//去重
					l++
				}
				for l < h && nums[h] == nums[h-1]{//去重
					h--
				}
				l++
				h--
			}else if sum < 0 {
				l++
			}else{
				h--
			}
		}
	}
	return result
}


//快速排序
func quickSort(nums []int,l,h int){
	if l >= h {
		return
	}
	m := partition(nums,l,h)
	quickSort(nums,l,m-1)
	quickSort(nums,m+1,h)
}

func partition(nums []int,l,h int)int{
	cursor := nums[h]
	j := l
	for i:=l;i<h;i++{
		if nums[i] < cursor {
			nums[i],nums[j] = nums[j],nums[i]
			j++
		}
	}
	nums[j],nums[h] =nums[h],nums[j]
	return j
}
~~~

### 2、数组中的第K个最大元素

在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。[215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

示例 1:

输入: [3,2,1,5,6,4] 和 k = 2
输出: 5
示例 2:

输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
输出: 4
说明:

你可以假设 k 总是有效的，且 1 ≤ k ≤ 数组的长度。

解答：

解法1、可按快排排序，然后取第K个元素，时间复杂度O(n*LogN)，最差退化到O（$n^2$)，这是最普通的解法。

解法2、快排的思想是一次一次将数组里的数据分成两部分，利用这个，我们可以再一次次将数据分成两部分之后，判断第K大的元素出现在哪一部分，然后对那部分数据递归进行分组，最终找到第K个最大元素，也就是所谓的快速选择

解法3、构建小顶堆来解决该问题

代码实现

~~~go
func findKthLargest(nums []int, k int) int {
	l := len(nums)
	if l == 1 {
		return nums[0]
	}
	return quickSelect(nums,0,l-1,k)

}


func partition4(nums []int,s,e,r int) int {
		a := nums[r]
		nums[r],nums[s] = nums[s],nums[r]
		p := e
		for i := e;i>0;i--{
			if nums[i] < a {
				nums[i],nums[p] = nums[p],nums[i]
				p--
			}
		}
		nums[p],nums[s] = nums[s],nums[p]
		return p

}

func quickSelect(nums []int,s,e,k int)int{
	if s == e {
		return nums[s]
	}
	r := s + rand.Intn(e-s)
	p := partition4(nums,s,e,r)
	if p == k-1 {
		return nums[p]
	} else if p > k-1 {
		return quickSelect(nums,0,p-1,k)
	}else {
		return quickSelect(nums,p+1,e,k)
	}

}
~~~

### 3、两数之和

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例:

l给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]

要求时间复杂度不超过O(n)

~~~go

~~~

