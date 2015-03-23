
#
礼品表 礼品库存表
    
gift    1======1    gift_stock

gift    1======N    gift_coupon

领取：gift_stock减一，gift_coupon[i]状态变成已领取,返回gift_coupon[i]信息。


业务校验：
1. 优惠券类型
2. gift状态正常（可理解成是否对外显示）
3. gift未过期
4. 用户是否到达领取上限
5. 积分金币是否足够
6. 领取逻辑



