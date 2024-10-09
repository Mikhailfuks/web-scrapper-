package main

import ("fmt""io/ioutil""log"net/http"strings"

)

func main() {
 // Create a new colly collector.
 collector := colly.NewCollector(
  colly.AllowedDomains("example.com"), // Allow scraping from example.com
 )

 // On every a element, print the href and text.
 collector.OnHTML("a[href]", func(e *colly.HTMLElement) {
  link := e.Attr("href")
  text := e.Text
  fmt.Printf("Link: %s - Text: %s\n", link, text)
 })

 // On every h1 element, print the text.
 collector.OnHTML("h1", func(e *colly.HTMLElement) {
  text := e.Text
  fmt.Printf("Heading: %s\n", text)
 })

 // Start scraping from the example.com website.
 err := collector.Visit("https://www.example.com")
 if err != nil {
  log.Fatal(err)
 }

 // Scrape all links in the website.
 collector.OnHTML("a[href]", func(e *colly.HTMLElement) {
  link := e.Attr("href")
  // Only scrape links that start with "https://"
  if !strings.HasPrefix(link, "https://") {
   return
  }
  // Visit the link
  collector.Visit(link)
 }

 // Scrape the content of a specific page.
 collector.OnHTML("div.content", func(e *colly.HTMLElement) {
  content := e.Text
  fmt.Printf("Content: %s\n", content)
 }

 // Download the HTML content of a specific page.
 collector.OnHTML("div.download", func(e *colly.HTMLElement) {
  url := e.Attr("href")
  // Download the HTML content of the URL
  resp, err := http.Get(url)
  if err != nil {
   log.Fatal(err)
  }
  defer resp.Body.Close()
  body, err := ioutil.ReadAll(resp.Body)
  if err != nil {
   log.Fatal(err)
  }
  fmt.Printf("Downloaded HTML: %s\n", string(body))
 }

 // Start scraping.
 collector.Wait()
}
