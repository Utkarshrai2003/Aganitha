import requests
import csv
import argparse
import re
from typing import List, Dict, Optional

# Constants
PUBMED_API_URL = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi"
PUBMED_SUMMARY_URL = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi"
PUBMED_DETAILS_URL = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi"

# Function to fetch PubMed paper IDs
def fetch_paper_ids(query: str) -> List[str]:
    params = {
        "db": "pubmed",
        "term": query,
        "retmode": "json",
        "retmax": 10  # Adjust as needed
    }
    response = requests.get(PUBMED_API_URL, params=params)
    response.raise_for_status()
    data = response.json()
    return data.get("esearchresult", {}).get("idlist", [])

# Function to fetch paper details
def fetch_paper_details(paper_id: str) -> Optional[Dict]:
    params = {
        "db": "pubmed",
        "id": paper_id,
        "retmode": "xml"
    }
    response = requests.get(PUBMED_DETAILS_URL, params=params)
    if response.status_code != 200:
        return None
    return response.text  # Returns raw XML (parsing required)

# Function to extract non-academic authors
def extract_non_academic_authors(text: str) -> List[str]:
    non_academic_authors = []
    company_keywords = ["pharma", "biotech", "inc", "corp", "gmbh", "s.a.", "llc"]
    
    for line in text.split("\n"):
        if any(keyword in line.lower() for keyword in company_keywords):
            match = re.search(r"<Author>(.*?)</Author>", line)
            if match:
                non_academic_authors.append(match.group(1))
    
    return non_academic_authors

# Main function to fetch and process papers
def get_papers(query: str) -> List[Dict]:
    paper_ids = fetch_paper_ids(query)
    results = []
    
    for paper_id in paper_ids:
        details = fetch_paper_details(paper_id)
        if details:
            non_academic_authors = extract_non_academic_authors(details)
            if non_academic_authors:
                results.append({
                    "PubmedID": paper_id,
                    "Title": "Unknown",  # Title parsing required
                    "Publication Date": "Unknown",  # Date parsing required
                    "Non-academic Author(s)": ", ".join(non_academic_authors),
                    "Company Affiliation(s)": "Unknown",  # Affiliation parsing required
                    "Corresponding Author Email": "Unknown"  # Email parsing required
                })
    return results

# Function to save results to CSV
def save_to_csv(results: List[Dict], filename: str):
    with open(filename, mode='w', newline='') as file:
        writer = csv.DictWriter(file, fieldnames=results[0].keys())
        writer.writeheader()
        writer.writerows(results)

# CLI Setup
def main():
    parser = argparse.ArgumentParser(description="Fetch research papers from PubMed")
    parser.add_argument("query", help="Search query for PubMed")
    parser.add_argument("-f", "--file", help="Output CSV filename", default=None)
    parser.add_argument("-d", "--debug", action="store_true", help="Enable debug mode")
    args = parser.parse_args()
    
    results = get_papers(args.query)
    
    if args.file:
        save_to_csv(results, args.file)
        print(f"Results saved to {args.file}")
    else:
        for result in results:
            print(result)
    
if __name__ == "__main__":
    main()
