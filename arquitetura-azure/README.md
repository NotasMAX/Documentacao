# Arquitetura alvo de nuvem — NotasMAX

Este documento apresenta a arquitetura alvo de desenvolvimento do NotasMAX, pensada para desenvolvimento e demonstração com serviços gratuitos ou recursos que podem ser cobertos pelo crédito Azure for Students.

## Arquitetura alvo de desenvolvimento

``` mermaid
flowchart TB
    subgraph clients["Clientes"]
        browser["Navegador"]
        mobile["Aplicativo mobile"]
    end

    subgraph azureDev["Azure for Students"]
        swaDev["Static Web Apps Free"]
        subgraph vnetDev["VNet de desenvolvimento"]
            acaDev["Container Apps Workload profiles"]
            apiDev["API Consumption, mínimo 0 réplicas"]
            natDev["NAT Gateway e IPv4 fixo"]
            apiDev --> natDev
        end
        keyVaultDev["Key Vault"]
        storageDev["Azure Storage Account\nBlob Storage privado\nAPI: identidade gerenciada / RBAC"]
        acaDev -. "identidade gerenciada" .-> keyVaultDev
    end

    subgraph githubDev["GitHub"]
        actionsDev["GitHub Actions"]
        ghcrDev["GHCR privado"]
        actionsDev -->|publica frontend| swaDev
        actionsDev -->|publica imagem| ghcrDev
    end

    atlasDev[("MongoDB Atlas Free M0")]

    browser -->|HTTPS| swaDev
    browser -->|HTTPS API| apiDev
    mobile -->|HTTPS API| apiDev
    natDev -->|IPv4 fixo no allowlist| atlasDev
    apiDev -. "identidade gerenciada / RBAC" .-> storageDev
    natDev -->|firewall: IPv4 NAT| storageDev
    ghcrDev -->|pull privado autenticado| apiDev
    actionsDev -->|atualiza API| acaDev


```


| Camada | Serviço | Uso |
|---|---|---|
| Frontend | Azure Static Web Apps Free | Hospedar o frontend React/Vite por HTTPS; inclui até 100 GB de largura de banda por assinatura/mês. |
| API | Azure Container Apps, ambiente Workload profiles e perfil Consumption | Executar a API Node.js/Express com mínimo de réplicas igual a zero. |
| Imagem da API | GitHub Container Registry (GHCR), repositório privado | Armazenar a imagem do container; não usar Azure Container Registry. |
| Banco | MongoDB Atlas Free (M0) | Armazenar dados de desenvolvimento e demonstração. |
| Armazenamento de arquivos | Azure Storage Account com Blob Storage e containers privados | Armazenar arquivos e objetos da aplicação; a API acessa com identidade gerenciada e RBAC. Configurar o firewall para negar por padrão e permitir o IPv4 NAT; desabilitar acesso anônimo e Shared Key. |
| Rede de saída | VNet e NAT Gateway Standard com IPv4 público estático | Manter o IP de saída da API e incluí-lo na lista de acesso do Atlas Free. O NAT Gateway StandardV2 ainda não é compatível com essa integração do Container Apps. |
| Segredos | Azure Key Vault e identidade gerenciada atribuída pelo sistema | Permitir à API ler segredos; guardar separadamente o token de leitura do GHCR. |

O navegador e o aplicativo mobile acessam a API por HTTPS. A API alcança o Atlas pela saída da VNet no NAT Gateway. O Atlas Free aceita acesso pela lista de IPs e não oferece Private Link; nesta arquitetura, o IPv4 fixo do NAT é a restrição de rede usada para o banco.

O MongoDB Atlas armazena os dados estruturados. Os arquivos e objetos ficam no Azure Blob Storage, em containers privados, sem acesso anônimo ou Shared Key; somente a API os acessa usando sua identidade gerenciada, com a função Storage Blob Data Contributor restrita ao container necessário. No alvo de desenvolvimento, o firewall da conta de armazenamento nega por padrão e permite o IPv4 fixo do NAT; navegadores e dispositivos não acessam o Blob Storage diretamente.

O Atlas Free tem capacidade e opções limitadas: 512 MB de armazenamento, disponibilidade em um conjunto restrito de regiões e ausência de Private Link, auditoria do banco e chaves de criptografia gerenciadas pelo cliente. Por isso, esta arquitetura atende ao cenário de desenvolvimento e demonstração.

O GitHub Actions publica a imagem privada da API no GHCR. O Azure Container Apps baixa a imagem usando um Personal Access Token (classic) do GitHub com escopo `read:packages`. Essa credencial fica no Key Vault, é referenciada pela configuração de registro do Container Apps e pode ser lida pela identidade gerenciada do aplicativo. A permissão de publicação da automação e a credencial de leitura usada pelo aplicativo são separadas.

A API pode escalar a zero quando não há réplicas. Isso reduz a cobrança de computação nesse intervalo, mas não remove o custo do NAT Gateway e do IPv4 estático, que continuam alocados.

Como essa configuração pode levar a API a zero réplicas, a primeira chamada após um período sem tráfego pode ter atraso de inicialização. Para uma apresentação em horário definido, configure uma réplica mínima antes de começar; a réplica ociosa pode gerar cobrança e consumir a franquia mensal. O IP fixo do NAT continua igual e não exige alterar a lista de acesso do Atlas.

### Limitação da conta Azure para estudantes

A oferta Azure for Students publicada pela Microsoft inclui US$ 100 em crédito por 12 meses e uma quantidade limitada de serviços gratuitos. A oferta lista Azure Blob Storage entre os serviços disponíveis, mas capacidade armazenada, operações, transferência e recursos de rede podem gerar cobrança e consumir crédito; a presença do serviço na oferta não torna qualquer configuração gratuita. Os demais serviços pagos também consomem o crédito. Quando o crédito acaba ou o período termina, os serviços podem ser desativados se a assinatura não for renovada ou convertida para pagamento conforme o uso. Assinaturas gratuitas, incluindo a de estudantes, não aceitam pedidos de aumento de quota; quotas maiores exigem a conversão para pagamento conforme o uso.

A oferta é destinada a educação ou ensino, pesquisa não comercial e atividades de desenho, desenvolvimento, teste e demonstração de software nesses contextos. O saldo e a data de renovação da assinatura individual não foram consultados; podem ser conferidos no [portal Azure Sponsorships](https://www.microsoftazuresponsorships.com/).

### Custo semanal estimado — desenvolvimento

| Item | Custo de referência | Observações |
|---|---:|---|
| Azure Static Web Apps Free | US$ 0 | Até 100 GB de largura de banda por assinatura/mês; acima da quota, o site pode deixar de ser servido. O plano Free não inclui SLA. |
| Azure Container Apps Consumption | US$ 0 dentro da franquia mensal | Inclui até 180.000 vCPU-segundos, 360.000 GiB-segundos e 2 milhões de solicitações por assinatura/mês. Sem réplicas, não há cobrança de computação da aplicação. |
| MongoDB Atlas Free | US$ 0 | Tem limites de armazenamento e recursos e não inclui Private Link. |
| Azure Blob Storage | Variável por região e uso | Depende de capacidade, operações, transferência e configuração de redundância. Não há volume nem padrão de acesso definidos para estimar um valor fixo. |
| NAT Gateway Standard e um IPv4 público estático | **Aproximadamente US$ 8,40/semana** | Referência calculada como 168 horas × (US$ 0,045/h para NAT + US$ 0,005/h para IPv4); confirme as tarifas da região e da oferta no calculador Azure. |
| Recursos gerenciados de rede do Container Apps | Variável por região e configuração | A VNet própria pode gerar cobrança de balanceador de carga e endereços IP gerenciados pelo ambiente. O ambiente externo também usa um IP público de entrada. |

O subtotal de referência já calculado para desenvolvimento é **aproximadamente US$ 8,40 por semana** para o NAT e um IPv4, antes do processamento de dados, da transferência de saída, do Blob Storage e dos recursos gerenciados de rede do Container Apps. Isso equivale a cerca de US$ 36,40 por mês médio. Se não houvesse nenhum outro consumo, US$ 100 cobririam cerca de 11,9 semanas apenas desse par NAT/IP; o custo efetivo do ambiente pode reduzir esse período.

O valor de US$ 8,40 é uma estimativa aritmética para planejamento baseada nas tarifas de referência indicadas, não uma cotação atual nem o custo total do ambiente. As páginas de preço do [NAT Gateway](https://azure.microsoft.com/en-us/pricing/details/azure-nat-gateway/), de [endereços IP](https://azure.microsoft.com/en-us/pricing/details/ip-addresses/) e do [Blob Storage](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/) variam por região e oferta; o calculador deve ser consultado antes da criação dos recursos. O NAT também cobra dados processados, e a transferência de saída pode ser cobrada à parte. O Container Apps em VNet própria pode criar endereços IP e balanceadores de carga cobrados separadamente; verifique os recursos gerenciados efetivamente criados. A VNet básica não tem custo próprio; NAT, IPv4, recursos gerenciados, Key Vault, monitoramento, armazenamento, recursos privados e consumo da API além das franquias podem usar o crédito.

## Referências oficiais

### Microsoft Azure

- [Serviços e créditos do Azure for Students](https://learn.microsoft.com/en-us/azure/education-hub/about-azure-for-students)
- [Azure for Students: perguntas frequentes e uso permitido](https://learn.microsoft.com/en-us/azure/education-hub/faq)
- [Assinatura Azure for Students desativada por fim do crédito ou do período](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/azurestudents-subscription-disabled)
- [Limites de serviço e quotas de assinatura Azure](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits)
- [Preços do Azure Static Web Apps](https://azure.microsoft.com/en-us/pricing/details/app-service/static/)
- [Quotas do Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/static-web-apps/quotas)
- [Token de implantação do Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/static-web-apps/deployment-token-management)
- [Cobrança do Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/billing)
- [Escalonamento e réplicas mínimas no Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/scale-app)
- [Rede virtual própria, NAT Gateway e recursos de rede cobrados no Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/custom-virtual-networks)
- [Preços do Azure Container Apps](https://azure.microsoft.com/en-us/pricing/details/container-apps/)
- [Segredos do Azure Container Apps com Key Vault e identidade](https://learn.microsoft.com/en-us/azure/container-apps/manage-secrets)
- [Preços do Azure NAT Gateway](https://azure.microsoft.com/en-us/pricing/details/azure-nat-gateway/)
- [Preços de endereços IP Azure](https://azure.microsoft.com/en-us/pricing/details/ip-addresses/)
- [Preços do Azure Blob Storage](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/)
- [Segurança da conta Azure Storage](https://learn.microsoft.com/en-us/azure/storage/common/secure-storage)
- [Regras de firewall e acesso de rede do Azure Storage](https://learn.microsoft.com/en-us/azure/storage/common/storage-network-security)
- [Autorização Blob com identidade Microsoft Entra](https://learn.microsoft.com/en-us/azure/storage/blobs/authorize-access-azure-active-directory)

### MongoDB

- [Regiões Microsoft Azure disponíveis para Atlas](https://www.mongodb.com/docs/atlas/reference/microsoft-azure/)
- [Limitações do Atlas Free](https://www.mongodb.com/docs/atlas/reference/free-shared-limitations/)

### GitHub

- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Publicar imagens GHCR privadas no Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/github-actions)
- [Autenticação OpenID Connect no Azure a partir do GitHub Actions](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-azure)
