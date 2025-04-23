---
date:       2016-03-24
title:      四则运算计算24点
slug:       calc-24-points
author:     ryan
tags:       [php, calc24]
comments:   true
weight:     560324
---

```php
# 运行环境：PHP 5.3+

/* 四个数字, 用四则运算和括号得到24 */
function calc24($a, $b, $c, $d)
{
    $nums = func_get_args();
    assert (is_int($a) && is_int($b)
            && is_int($c) && is_int($d));
    $lists = arrange_nums($a, $b, $c, $d);
    $exprs = arrange_ops();
    foreach ($lists as $i => $nums) {
        //echo $i + 1, ": ", implode(' ', $nums), "\n";
        foreach ($exprs as $j => $expr) {
            $evaluation = vsprintf($expr, $nums);
            try {
                $result = @eval('return ' . $evaluation . ';');
            } catch (Exception $e) { //Division by zero
                continue;
            }
            //echo $j + 1, ": ", $evaluation . ' = ' . $result, "\n";
            if (abs($result - 24) < 0.000001) {
                return $evaluation . ' = 24';
            }
        }
    }
    return false;
}

/* 列出四个数字可能的位置 */
function arrange_nums($a, $b, $c, $d)
{
    $result = array();
    for ($i = 0; $i < 4; $i ++) {
        $nums = array(0, 0, 0, 0);
        $nums[$i] = $a;
        for ($j = 0; $j < 4; $j ++) {
            if ($j === $i) {
                continue;
            }
            $nums[$j] = $b;
            for ($k = 0; $k < 4; $k ++) {
                if ($k === $i || $k === $j) {
                    continue;
                }
                $nums[$k] = $c;
                $nums[6 - $i - $j - $k] = $d;
                $result[] = $nums;
            }
        }
    }
    return $result;
}

/* 列出所有可能运算以及结合次序 */
function arrange_ops()
{
    static $result = array();
    if (count($result) > 0) {
        return $result;
    }
    $ops = array('+', '-', '*', '/');
    foreach ($ops as $op1) {
        foreach ($ops as $op2) {
            foreach ($ops as $op3) {
                $result[] = '((%d' .$op1. '%d)' .$op2. '%d)' .$op3. '%d';
                $result[] = '(%d' .$op1. '%d)' .$op2. '(%d' .$op3. '%d)';
                $result[] = '(%d' .$op1. '(%d' .$op2. '%d))' .$op3. '%d';
                $result[] = '%d' .$op1. '((%d' .$op2. '%d)' .$op3. '%d)';
                $result[] = '%d' .$op1. '(%d' .$op2. '(%d' .$op3. '%d))';
            }
        }
    }
    return $result;
}

/* 一些较难但无解的题目 */
echo calc24(5, 8, 68, 2), "\n";
echo calc24(3, 3, 8, 8), "\n";
echo calc24(5, 5, 7, 8), "\n";
echo calc24(12, 23, 45, 2), "\n";
echo calc24(3, 3, 7, 7), "\n";
echo calc24(13, 13, 1, 7), "\n";
echo calc24(5, 5, 5, 1), "\n";
echo calc24(1, 6, 11, 13), "\n";
echo "\n";

/* 19xx中有多少无解 */
for ($c = 1; $c < 10; $c ++) {
    for ($d = 1; $d < 10; $d ++) {
        $result = calc24(1, 9, $c, $d);
        if ($result === false) {
            echo "19", $c, $d, "  ";
        }
    }
}
echo "\n";
```
