---
date:       2012-05-15
title:      Python多进程记录日志
slug:       python-mulpti-logging
author:     ryan
tags:       [python, logging, mulpti-process]
weight:     620515
---

## 问题描述

用gevent（或封装了gevent的gunicore）启动python进程，会出现多个独立进程同时写一个日志文件， 可以观察到有日志部分丢失：一个进程日志没写完，另一个进程把日志覆盖在同一行的后面；有些日志甚至完全丢失。 用mlogging包可以解决多进程写日志的问题，没有发现不完整的日志，是否丢失日志有待进一步检测。

## 解决方案与代码

下面是一个在python程序中记录重要信息，以便以后解析统计的函数

```python
# -*- coding: utf-8 -*-

import os, os.path
import logging
from logging.handlers import TimedRotatingFileHandler
from mlogging import TimedRotatingFileHandler_MP
from functools import partial
from datetime import datetime


DEBUG  = False
LOG_LEVEL = 'INFO'
LOG_FILENAME = 'access'


def patch_logger(logger, level = 'NOTSET', is_debug = False):
    if is_debug:
        logger.setLevel(logging.DEBUG)
        def print_log(level, message, *args, **kwargs):
            dt = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
            print dt, level, message
        setattr(logger, '_log', print_log) #输出到屏幕上
    else:
        #设置级别，DEBUG/INFO/WARNING/ERROR/CRITICAL
        logger.setLevel(getattr(logging, level))
        #有些Python版本会报错KeyError，找不到clientip或user，这里用一个短横(-)做默认值
        extra = {'clientip':'-', 'user':'-'}
        #exc_info是出错时的Debug详细回溯信息，这里禁止记录，只记录错误信息这一行
        setattr(logger, '_log', partial(logger._log, exc_info=False, extra=extra))
    return logger


def create_handler(handler_class = logging.FileHandler, filename = 'access',
                    logging_dir = './logs', filter = None):
    if not os.path.exists(logging_dir):
        os.mkdir(logging_dir)
    logging_file = os.path.join(logging_dir, filename + '.log')
    handler = handler_class(logging_file, 'midnight', 1)
    #设置日志格式，固定宽度便于解析，asctime时间格式
    handler.setFormatter(logging.Formatter(
        '%(asctime)s %(levelname)s %(name)s %(message)s',
        datefmt = '%Y-%m-%d %H:%M:%S'
    ))
    if isinstance(handler, TimedRotatingFileHandler):
        handler.suffix = '%Y%m%d'
        handler.extMatch = r"^20\d{6}$"
    if isinstance(filter, logging.Filter):
        handler.addFilter(filter) #加载过滤器
    return handler


logger = logging.getLogger(__name__)
logger = patch_logger(logger, level = LOG_LEVEL, is_debug = DEBUG)
log_handler = create_handler(TimedRotatingFileHandler_MP, filename = LOG_FILENAME)
logger.addHandler(log_handler)



if __name__ == '__main__':
    logger.debug('低级别的DEBUG，不会记录。')
    logger.info('哈哈哈，这才是我想要的信息，请记下来。')
```
