# Weather-API

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpHeaders;
import org.springframework.web.client.RestTemplate;

@Configuration
public class WeatherApiConfig {

    @Value("${rapidapi.client.id}")
    private String clientId;

    @Value("${rapidapi.client.secret}")
    private String clientSecret;

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    public HttpHeaders rapidApiHeaders() {
        HttpHeaders headers = new HttpHeaders();
        headers.set("x-rapidapi-host", "weatherapi-com.p.rapidapi.com");
        headers.set("x-rapidapi-key", clientId); // Using client id as the API key for simplicity
        return headers;
    }
}

Next, let's create a service to interact with the Weather API:

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class WeatherApiService {

    private final RestTemplate restTemplate;
    private final HttpHeaders rapidApiHeaders;

    @Autowired
    public WeatherApiService(RestTemplate restTemplate, HttpHeaders rapidApiHeaders) {
        this.restTemplate = restTemplate;
        this.rapidApiHeaders = rapidApiHeaders;
    }

    public String getForecastSummaryByLocationName(String locationName) {
        String url = "https://weatherapi-com.p.rapidapi.com/forecast.json?q=" + locationName;
        HttpEntity<String> entity = new HttpEntity<>(rapidApiHeaders);
        return restTemplate.postForObject(url, entity, String.class);
    }

    public String getHourlyForecastByLocationName(String locationName) {
        String url = "https://weatherapi-com.p.rapidapi.com/forecast.json?q=" + locationName + "&days=1";
        HttpEntity<String> entity = new HttpEntity<>(rapidApiHeaders);
        return restTemplate.postForObject(url, entity, String.class);
    }
}


This service class contains methods to retrieve the forecast summary and hourly forecast details for a given location.

Finally, let's create a controller to expose these APIs:

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class WeatherController {

    private final WeatherApiService weatherApiService;

    @Autowired
    public WeatherController(WeatherApiService weatherApiService) {
        this.weatherApiService = weatherApiService;
    }

    @GetMapping("/weather/forecast-summary")
    public String getForecastSummary(@RequestParam String locationName) {
        return weatherApiService.getForecastSummaryByLocationName(locationName);
    }

    @GetMapping("/weather/hourly-forecast")
    public String getHourlyForecast(@RequestParam String locationName) {
        return weatherApiService.getHourlyForecastByLocationName(locationName);
    }
}
