package mouse_operation;
import org.openqa.selenium.*;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import io.github.bonigarcia.wdm.WebDriverManager;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.StandardCopyOption;
import java.time.Duration;
import java.util.List;
import java.util.Set;

public class Project_example {

    public static void main(String[] args) throws IOException, InterruptedException {

        WebDriverManager.chromedriver().setup();
        WebDriver driver = new ChromeDriver();
        driver.manage().window().maximize();

        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));

        driver.get("https://www.flipkart.com/");

        // Close Login Popup
        try {
            WebElement closeBtn = wait.until(
                    ExpectedConditions.elementToBeClickable(By.xpath("//button[text()='✕']")));
            closeBtn.click();
        } catch (Exception e) {
            System.out.println("Login popup not displayed.");
        }

        // Search Bluetooth Speakers
        WebElement searchBox = wait.until(
                ExpectedConditions.visibilityOfElementLocated(By.name("q")));
        searchBox.sendKeys("Bluetooth Speakers");
        searchBox.submit();

        // Apply Brand Filter → boAt
        wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//div[contains(text(),'Brand')]")));
        WebElement boatBrand = wait.until(
                ExpectedConditions.elementToBeClickable(By.xpath("//div[@title='boAt']")));
        boatBrand.click();

        // Apply Rating Filter → 4★ & above
        WebElement ratingFilter = wait.until(
                ExpectedConditions.elementToBeClickable(By.xpath("//div[text()='4★ & above']")));
        ratingFilter.click();

        // Sort → Price Low to High
        WebElement sortLowToHigh = wait.until(
                ExpectedConditions.elementToBeClickable(By.xpath("//div[text()='Price -- Low to High']")));
        sortLowToHigh.click();

        Thread.sleep(3000);

        // Open 1st product in new tab
        WebElement firstProduct = wait.until(
                ExpectedConditions.elementToBeClickable(By.xpath("(//a[contains(@href,'/p/')])[1]")));

        String clickLink = Keys.chord(Keys.CONTROL, Keys.ENTER);
        firstProduct.sendKeys(clickLink);

        // Switch to new tab
        Set<String> handles = driver.getWindowHandles();
        String mainTab = driver.getWindowHandle();

        for (String handle : handles) {
            if (!handle.equals(mainTab)) {
                driver.switchTo().window(handle);
            }
        }

        // Check Available Offers
        try {
            WebElement offersSection = driver.findElement(By.xpath("//span[contains(text(),'Available offers')]"));
            if (offersSection.isDisplayed()) {

                List<WebElement> offersList = driver.findElements(By.xpath("//div[contains(@class,'_16eBzU')]"));
                System.out.println("Number of available offers: " + offersList.size());
            }
        } catch (Exception e) {
            System.out.println("Available offers section not found.");
        }

        // SCENARIO CHECK
        try {
            WebElement addToCartBtn = driver.findElement(By.xpath("//button[contains(text(),'Add to cart')]"));

            if (addToCartBtn.isDisplayed() && addToCartBtn.isEnabled()) {

                System.out.println("Product is available.");

                addToCartBtn.click();

                wait.until(ExpectedConditions.urlContains("cart"));

                takeScreenshot(driver, "cart_result.png");

            } else {
                System.out.println("Product unavailable — could not be added to cart.");
                takeScreenshot(driver, "result.png");
            }

        } catch (Exception e) {

            System.out.println("Product unavailable — could not be added to cart.");
            takeScreenshot(driver, "result.png");
        }

        Thread.sleep(3000);
        driver.quit();
    }

    public static void takeScreenshot(WebDriver driver, String fileName) throws IOException {

        File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
        File dest = new File(fileName);
        Files.copy(src.toPath(), dest.toPath(), StandardCopyOption.REPLACE_EXISTING);

        System.out.println("Screenshot saved as: " + fileName);
    }
}
