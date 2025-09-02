`from DrissionPage import WebPage,ChromiumOptions
import time

def get_xiaohongshu_search_results(keyword):
    """
    使用 DrissionPage 获取小红书搜索结果页数据。

    Args:
        keyword (str): 搜索关键词。

    Returns:
        list: 包含笔记信息的字典列表。
    """
    print("正在启动浏览器...")
    # 初始化一个 WebPage 对象，这将启动一个 Chrome 浏览器
    # headless=False 表示以可视模式运行，方便调试
    co = ChromiumOptions()
    co.set_user_data_path('./chrome_data')
    co.set_local_port(9223)
    co.set_argument('--disable-blink-features=AutomationControlled')  # 避免被检测
    page = WebPage(chromium_options= co)
    # 小红书搜索结果页 URL 格式
    search_url = f'https://www.xiaohongshu.com/search_result/?keyword={keyword}'
    print(f"正在访问搜索页面: {search_url}")
    try:
        page.get(search_url)

        # 增加等待时间，确保页面内容完全加载
        # 这里可以使用 page.wait.ele_loaded() 等更精确的等待方法，但简单的 time.sleep 也可以
        print("等待页面加载...")
        time.sleep(5)

        # 可能会遇到登录弹窗，可以尝试关闭它
        try:
            close_button = page.ele('css:i[class*="icon-close"]')
            if close_button:
                close_button.click()
                print("已关闭登录弹窗。")
        except:
            pass  # 如果没有找到弹窗，则忽略

        # 定位笔记列表容器，小红书笔记通常在特定的 div 容器内
        # 这里需要根据最新的网页结构来调整选择器
        for i in range(5):
            page.scroll.down(300)
            time.sleep(1)
        note_list_container = page.ele('.feeds-container', timeout=10)
        if not note_list_container:
            print("未找到笔记列表容器，可能页面结构已改变或仍在加载中。")
            # 尝试滚动页面以触发加载
            return []

        # 获取所有笔记项
        notes = note_list_container.eles('css:.note-item')
        if not notes:
            print("未找到任何笔记。")
            return []

        results = []
        print(f"找到 {len(notes)} 条笔记，正在提取数据...")

        for note in notes:
            try:
                # 提取标题
                title_ele = note.ele('css:.title')
                title = title_ele.text if title_ele else '无标题'

                # 提取作者
                author_ele = note.ele('.name')
                author = author_ele.text if author_ele else '未知作者'

                # 提取链接
                link_ele = note.ele('tag:a')
                link = link_ele.attr('href') if link_ele else '无链接'
                if link and not link.startswith('http'):
                    link = 'https://www.xiaohongshu.com' + link

                results.append({
                    'title': title,
                    'author': author,
                    'link': link
                })
            except Exception as e:
                print(f"提取单条笔记数据时出错: {e}")
                continue
        print(results)
        return results


    except Exception as e:
        print(f"发生错误: {e}")
        return []
    finally:
        # 确保浏览器被关闭
        print("任务完成，正在关闭浏览器...")
        page.quit()

if __name__ == '__main__':
    search_keyword = '美食'
    note_data = get_xiaohongshu_search_results(search_keyword)

    if note_data:
        print("\n--- 搜索结果 ---")
        for i, note in enumerate(note_data):
            print(f"{i+1}. 标题: {note['title']}")
            print(f"   作者: {note['author']}")
            print(f"   链接: {note['link']}")
            print("-" * 20)
    else:
        print("未能成功获取数据。")`