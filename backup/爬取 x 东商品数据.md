```python
import time
import random
from DrissionPage import ChromiumPage,ChromiumOptions

class JDProductSpider:
    def __init__(self):
        """
        初始化爬虫
        """
        co = ChromiumOptions()
        co.set_user_data_path('./chrome_data')
        co.set_local_port(9222)
        co.set_argument('--start-maximized')  # 最大化窗口
        co.set_argument('--disable-blink-features=AutomationControlled')  # 避免被检测
        self.page = ChromiumPage(co)
        self.base_url = "https://search.jd.com/Search"

    def search_products(self, keyword, max_pages=1):
        """
        搜索商品并获取信息
        """
        products = []

        for page in range(1, max_pages + 1):
            print(f"正在爬取第 {page} 页...")

            # 构造搜索URL
            search_url = f"{self.base_url}?keyword={keyword}&page={page}"
            print(f"访问URL: {search_url}")
            self.page.get(search_url)

            # 等待页面加载完成
            print("等待页面加载...")
            time.sleep(random.uniform(3, 5))  # 增加等待时间

            # 检查页面标题和内容
            print(f"页面标题: {self.page.title}")
            # 打印部分页面内容用于调试
            print(f"页面部分HTML: {self.page.html[:500]}")

            # 滚动页面以触发懒加载
            print("滚动页面以加载更多内容...")
            for i in range(5):  # 增加滚动次数
                self.page.scroll.down(300)
                time.sleep(1)  # 增加每次滚动后的等待时间

            # 等待元素出现
            print("等待商品元素加载...")
            self.page.wait.eles_loaded('.gl-item', timeout=10)

            # 获取商品列表
            product_items = self.page.eles('.gl-item')
            print(f"找到 {len(product_items)} 个商品元素")

            # 如果没有找到商品，尝试其他选择器
            if len(product_items) == 0:
                print("尝试备用选择器...")
                product_items = self.page.eles('[data-sku]') or self.page.eles('.goods-item')
                print(f"备用选择器找到 {len(product_items)} 个商品元素")

            if len(product_items) == 0:
                print("警告：未找到任何商品元素")
                # 打印更多页面内容用于分析
                print("页面完整HTML结构（前1000字符）:")
                print(self.page.html[:1000])

            for i, item in enumerate(product_items):
                print(f"处理第 {i + 1} 个商品...")
                try:
                    product_info = self._extract_product_info(item)
                    if product_info:
                        products.append(product_info)
                        print(f"已添加商品，当前共 {len(products)} 个商品")
                    else:
                        print("未提取到有效商品信息")

                    # 限制处理数量以便调试
                    # if i >= 2:  # 只处理前3个商品进行测试
                    #     print("为调试目的，仅处理前3个商品")
                    #     break
                except Exception as e:
                    print(f"提取第 {i + 1} 个商品信息时出错: {e}")
                    continue

            time.sleep(random.uniform(2, 3))

        return products

    # 在 _extract_product_info 方法中添加更详细的调试信息
    def _extract_product_info(self, item):
        """
        提取单个商品信息
        """
        try:
            print("开始提取商品信息...")

            # 提取标题 - 处理title属性为空的情况
            title_ele = item.ele('.p-name p-name-type-2').ele('tag:a')
            print(f"标题元素: {title_ele}")
            if title_ele:
                print(f"title_ele 的 HTML 内容: {title_ele.html}")
                print(f"title_ele 的所有属性: {title_ele.attrs}")
            else:
                print("title_ele 元素未找到。请检查选择器。")


            title = ""
            if title_ele:
                # 按优先级尝试获取标题
                title = (title_ele.attr('title') or
                         title_ele.text or
                         "")
                # 如果还是空，尝试从em标签获取
                if not title:
                    em_title = title_ele.ele('tag:em')
                    if em_title:
                        title = em_title.text
                print(f"标题内容: '{title}'")


            # 提取价格
            price_ele = item.ele('.p-price i') or item.ele('.price') or item.ele('tag:strong')
            print(f"价格元素: {price_ele}")

            price = ""
            if price_ele:
                price = price_ele.text or price_ele.attr('data-price') or ""
            print(f"价格内容: '{price}'")


            # 提取链接
            link_ele = item.ele('.p-name p-name-type-2').ele('tag:a')
            print(f"链接元素: {link_ele}")

            link = ""
            if link_ele:
                link = link_ele.attr('href') or ""
                if link and not link.startswith('http'):
                    link = 'https:' + link
            print(f"商品链接: '{link}'")

            # 提取店铺信息 - 尝试更多选择器

            shop_ele = item.ele('.curr-shop hd-shopname')
            print(f"店铺元素: {shop_ele}")
            shop = ""
            if shop_ele:
                shop = shop_ele.text or shop_ele.attr('title') or ""
            print(f"店铺名称: '{shop}'")

            # 提取评论数 - 尝试更多选择器
            commit_ele = item.ele('.p-commit').ele('tag:a')
            print(f"评价元素: {commit_ele}")

            commit_count = ""
            if commit_ele:
                commit_count = (commit_ele.text or commit_ele.attr('title') or
                                "")
                print(f"评价数量: '{commit_count}'")


            # 构建结果
            result = {
                'title': title.strip() if title else "标题未知",
                'price': price.strip() if price else "价格未知",
                'link': link.strip() if link else "",
                'shop': shop.strip() if shop else "店铺未知",
                'commit_count': commit_count.strip() if commit_count else "评论未知"
            }

            # 检查是否获取到有效信息
            has_valid_info = any([
                result['title'] != "标题未知",
                result['price'] != "价格未知",
                result['shop'] != "店铺未知"
            ])

            if has_valid_info:
                print(f"成功提取商品: {result['title'][:30]}...")
                return result
            else:
                print("未提取到有效商品信息")
                return None

        except Exception as e:
            print(f"提取商品信息失败: {e}")
            import traceback
            traceback.print_exc()
            return None

    def save_to_file(self, products, filename="jd_products.txt"):
        """
        保存商品信息到文件

        Args:
            products (list): 商品信息列表
            filename (str): 保存的文件名
        """
        with open(filename, 'w', encoding='utf-8') as f:
            for product in products:
                f.write(f"标题: {product['title']}\n")
                f.write(f"价格: {product['price']}\n")
                f.write(f"店铺: {product['shop']}\n")
                f.write(f"评论数: {product['commit_count']}\n")
                f.write(f"链接: {product['link']}\n")
                f.write("-" * 50 + "\n")

        print(f"已保存 {len(products)} 条商品信息到 {filename}")

    def close(self):
        """
        关闭浏览器
        """
        self.page.quit()

def main():
    """
    主函数
    """
    # 创建爬虫实例
    spider = JDProductSpider()

    try:
        # 搜索关键词
        keyword = input("请输入要搜索的商品关键词: ")
        max_pages = int(input("请输入要爬取的页数(默认1页): ") or "1")

        # 开始爬取
        print(f"开始爬取京东 '{keyword}' 的商品信息...")
        products = spider.search_products(keyword, max_pages)

        # 显示结果
        print(f"\n共爬取到 {len(products)} 件商品:")
        for i, product in enumerate(products[:5], 1):  # 只显示前5个
            print(f"\n{i}. {product['title'][:50]}...")
            print(f"   价格: {product['price']}")
            print(f"   店铺: {product['shop']}")

        # 保存到文件
        if products:
            save_option = input("\n是否保存到文件? (y/n): ")
            if save_option.lower() == 'y':
                spider.save_to_file(products, f"jd_{keyword}_products.txt")

    except Exception as e:
        print(f"爬取过程中出现错误: {e}")
    finally:
        # 关闭浏览器
        spider.close()

if __name__ == "__main__":
    main()

``` 