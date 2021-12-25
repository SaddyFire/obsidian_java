```java
import com.tanhua.dubbo.api.RecommendUserApi;
import com.tanhua.dubbo.api.UserInfoApi;
import com.tanhua.model.domain.pojo.UserInfo;
import com.tanhua.model.domain.vo.TodayBestVo;
import com.tanhua.model.mongo.RecommendUser;
import com.tanhua.server.interceptor.UserHolder;
import com.tanhua.server.service.TanhuaService;
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.stereotype.Service;

import java.lang.reflect.InvocationTargetException;

/**
 * @author SaddyFire
 * @date 2021/12/25
 * @TIME:19:51
 */
@Service
public class TanhuaServiceImpl implements TanhuaService {

    @DubboReference
    private UserInfoApi userInfoApi;

    @DubboReference
    private RecommendUserApi recommendUserApi;


    @Override
    public TodayBestVo todayBest(String token) throws InvocationTargetException, IllegalAccessException {
        Long userId = UserHolder.getUserId();
        //根据用户id 到 mongodb查询 最佳推荐 的人
        RecommendUser recommendUser = recommendUserApi.getBest(userId);
        //若没有
        if(recommendUser == null){
            recommendUser = new RecommendUser();
            recommendUser.setUserId(1l);
            recommendUser.setScore(99d);
        }
        UserInfo userInfo = userInfoApi.getUserInfo(recommendUser.getUserId());
        TodayBestVo todayBestVo = TodayBestVo.init(userInfo, recommendUser);
        return todayBestVo;
    }
}
```