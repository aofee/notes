## php相关

### 正则


```
email
([a-zA-Z0-9.-]+)@([a-zA-Z0-9.-]+)\.([a-zA-Z]+)

```


### 算法

1. 有一母牛，到4岁可生育，每年一头，所生均是一样的母牛，到15岁绝育，不再能生，20岁死亡，问n年后有多少头牛

```php

function ox($n)
{
    $num = 1;
    for ($i=1; $i<=$n; $i++)
    {
        if ($i>=4 && $i<15)
        {
            $num++;
            ox($n-$i);
        }else if ($i==20){
            $num--;
        }
    }
    return $num;
}

```

2. 快速排序

```php

function quick_sort($arr)
{
    if (count($arr) <= 1){
        return $arr;
    }
    $temp = $arr[0];

    $left = array();
    $right = array();

    for ($i=1;$i<count($arr);$i++)
    {
        if ($arr[$i] > $temp)
        {
            $right[] = $arr[$i];
        }else{
            $left[] = $arr[$i];
        }
    }
    $left = quick_sort($left);
    $right = quick_sort($right);
    return array_merge($left,array($temp),$right);
}

print_r(quick_sort([-3,5,31]));

```
3. 冒泡排序

```php
function bubble_sort($arr) {
    // 判断参数是否为数组，且不为空
    if (!is_array($arr) || empty($arr)) {
        return $arr;
    }
    // 循环需要冒泡的轮数
    for ($i = 1, $len = count($arr); $i < $len; $i++) {
        // 循环每轮需要比较的次数
        for ($j = 0; $j < $len - $i; $j++) {
            // 大的数，交换位置，往后挪
            if ($arr[$j] > $arr[$j + 1]) {
                $temp = $arr[$j + 1];
                $arr[$j + 1] = $arr[$j];
                $arr[$j] = $temp;
            }
        }
    }
    return $arr;
}
```

4. 二分查找

```php

function binary_search($arr, $limit)
{
    $len = count($arr);
    $low = 0;
    $high = $len - 1;

    while ($low <= $high)
    {
        $middle = intval(($low+$high)/2);
        if ($arr[$middle] > $limit)
        {
            $high = $middle - 1;
        } else if ($arr[$middle] < $limit) {
            $low = $middle + 1;
        } else {
            return $middle;
        }
    }
    return -1;
}

print_r(binary_search([1,4,6,21,43,54,65,76,87,98,102],98));

```