### 前言：

​		  *对查询出来的问题列表进行分页的操作，并在前端展示。*

​		*这里侧重在问题列表的分页实现的过程，所以问题列表的查询等逻辑就不过多撰述了。*

------

#### 第一步，封装需要传到页面的元素

```java
package com.chenxi.dto
import lombok.Data;
import java.util.ArrayList;
import java.util.List;

/**
 * @Author: Mr.Chen
 * @Description: 封装页面的元素，包括问题展示列表和分页的内容
 * @Date:Created in 18:38 2020/3/10
 */
@Data
public class PaginationDTO<T> {
    /**
     * 是否展示首页
    */
    private Boolean showFirstPage;
    /**
     * 是否展示上一页
     */
    private Boolean showPrevious;
    /**
     * 是否展示下一页
     */
    private Boolean showNext;
    /**
     * 是否展示最后一页
     */
    private Boolean showEndPage;
    /**
     * 总页数
     */
    private Integer totalPage;
    /**
     * 当前页码
     */
    private Integer page;
    //问题列表数据
    private List<T> data;
    //存放所有需要展示的页码，1，2，3，4...
    private List<Integer> pages;
}
```



#### 第二步，在PaginationDTO里构建分页方法

```java
 	/**
 	 * @param totalCount 问题条数
 	 * @param page 当前页码，初始值为1
     * @param pageSize 一页包含的问题条数，初始值为5
     * @return 返回第一个返回记录行的偏移量，初始量为0
     */
    public Integer getPagination(Integer totalCount, Integer page, Integer pageSize) {
        //获取总页数
        if (totalCount % pageSize == 0) {
            totalPage = totalCount / pageSize;
        } else {
            totalPage = totalCount / pageSize + 1;
        }
        //页码越界处理
        if (page < 1) {
            page = 1;
        }
        if (page > totalPage) {
            page = totalPage;
        }
        Integer offset = pageSize * (page - 1);
        //添加页码，展示7个
        pages = new ArrayList<>();
        pages.add(page);
        for (int i = 1; i <= 3; i++) {
            if (page - i > 0) {
                //每次都添加进集合的首部
                pages.add(0, page - i);
            }
            if (page + i <= totalPage) {
                //在集合的尾部追加数据
                pages.add(page + i);
            }
        }
        this.page = page;
        showPrevious = (page != 1);
        showNext = (!page.equals(totalPage));
        showFirstPage = !pages.contains(1);
        showEndPage = !pages.contains(totalPage);
        return offset;
    }
```



#### 第三步，创建Controller来调用

```java
import com.chenxi.community.dto.PaginationDTO;
import com.chenxi.community.dto.QuestionDTO;
import com.chenxi.community.service.NotificationsService;
import com.chenxi.community.service.QuestionService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * @Author: Mr.Chen
 * @Description: 首页控制器
 * @Date:Created in 14:20 2020/3/1
 */
@Controller
public class IndexController {

    @Autowired
    private QuestionService questionService;

    @GetMapping("/")
    //从前端获取当前选择的页码和页面容纳问题的大小
    public String index(@RequestParam(name = "page", defaultValue = "1") Integer page,
                        @RequestParam(name = "pageSize", defaultValue = "5") Integer pageSize,
                        Model model) {
        //展示首页问题列表，getPaginationDTOList里调用了getPagination()，并将数据封装到QuestionDTO返回
        PaginationDTO<QuestionDTO> paginationDTO = questionService.getPaginationDTOList("0", page, pageSize);
        model.addAttribute("pagination1", paginationDTO);
        return "index";
    }
}
```



#### 第四步，前端页面

```html
<!-- 分页核心代码 -->
<nav aria-label="Page navigation">
                <ul class="pagination pull-right">
                    <li th:if="${pagination1.showFirstPage}">
                        <a th:href="@{/(page=1)}" aria-label="Previous">
                            <span aria-hidden="true">&lt;&lt;</span>
                        </a>
                    </li>
                    <li th:if="${pagination1.showPrevious}">
                        <a th:href="@{/(page=${pagination1.page}-1)}" aria-label="Previous">
                            <span aria-hidden="true">&lt;</span>
                        </a>
                    </li>
                    <li th:each="page : ${pagination1.pages}" th:class="${pagination1.page == page} ? 'active' : ''">
                        <a th:href="@{/(page=${page})}" th:text="${page}"></a>
                    </li>
                    <li th:if="${pagination1.showNext}">
                        <a th:href="@{/(page=${pagination1.page}+1)}" aria-label="Next">
                            <span aria-hidden="true">&gt;</span>
                        </a>
                    </li>
                    <li th:if="${pagination1.showEndPage}">
                        <a th:href="@{/(page=${pagination1.totalPage})}" aria-label="Next">
                            <span aria-hidden="true">&gt;&gt;</span>
                        </a>
                    </li>
                </ul>
            </nav>
```



#### 最后，实现结果



<img src="/images/page.PNG" style="zoom:75%;" />

​		**结果分析：上图是第三页问题列表展示，用户点击'3'之后，首先数据会传到IndexController，数据为[page=3, pageSize=5]，然后通过QuestionService进行处理，包括问题查询，问题封装，异常处理等，然后返回一个PaginationDTO集合，再渲染到前端的页面，最终展示出来。**



*Thank you.*





